# Encrypted Storage — Jotai + Web Crypto API

> **Stack:** Next.js App Router · TypeScript · Jotai · Web Crypto API  
> **Purpose:** AES-GCM encrypted `localStorage` persistence for client-side auth state, with server-side key derivation, session-cookie sync, and SSR safety.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [File Structure](#2-file-structure)
3. [Crypto Utilities](#3-crypto-utilities)
4. [Server Action — Key Derivation](#4-server-action--key-derivation)
5. [Encrypted Storage Adapter](#5-encrypted-storage-adapter)
6. [Auth Atom](#6-auth-atom)
7. [Jotai Provider & Session Sync](#7-jotai-provider--session-sync)
8. [Data Flow](#8-data-flow)
9. [Security Model](#9-security-model)
10. [Environment Variables](#10-environment-variables)
11. [Usage Examples](#11-usage-examples)
12. [Production Hardening Checklist](#12-production-hardening-checklist)

---

## 1. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      SERVER (Node.js)                           │
│                                                                 │
│   STORAGE_ENCRYPTION_KEY  (env — never reaches the browser)    │
│          │                                                      │
│          ▼  PBKDF2 (100k iterations, SHA-256)                  │
│   getClientEncryptionKey()  →  AES-GCM raw bytes (base64)      │
│          │                                                      │
│          │  HTTPS  (Server Action response)                     │
└──────────┼──────────────────────────────────────────────────────┘
           │
┌──────────▼──────────────────────────────────────────────────────┐
│                      CLIENT (Browser)                           │
│                                                                 │
│   initCryptoKey(base64)  →  imports raw bytes as CryptoKey     │
│          │  non-extractable, in-memory only                     │
│          │                                                      │
│   encryptText / decryptText  (await deferred key promise)       │
│          │                                                      │
│   createEncryptedStorage  →  Jotai atomWithStorage adapter      │
│          │                                                      │
│   authAtom  →  unwrap()  →  React components                   │
│          │                                                      │
│   SessionSync  →  polls __ca_ux_i cookie                       │
└─────────────────────────────────────────────────────────────────┘
```

**Why this pattern?**

| Concern | Solution |
|---|---|
| Master key visible in JS bundle | Derived server-side; only raw AES bytes cross the wire |
| Plain-text tokens in `localStorage` | AES-GCM 256-bit encryption per write |
| `encryptText` called before key is ready | Deferred promise — calls queue, never throw |
| SSR hydration mismatch | SSR no-op storage + `unwrap` fallback to `initialState` |
| Stale auth after server-side logout | Cookie indicator polling via `SessionSync` |
| Async storage with sync components | `unwrap()` bridges async atom → sync |

---

## 2. File Structure

```
src/
├── app/
│   └── actions/
│       └── crypto.action.ts           # Server Action: PBKDF2 key derivation
├── lib/
│   ├── utils/
│   │   └── crypto.ts                  # AES-GCM encrypt/decrypt (deferred key)
│   └── store/
│       ├── auth.atom.ts               # Jotai auth atom (encrypted persistence)
│       └── encrypted-storage.ts       # Custom Jotai storage adapter
└── providers/
    └── jotai-provider.tsx             # <Provider> + <CryptoKeyInit> + <SessionSync>
```

---

## 3. Crypto Utilities

**File:** `lib/utils/crypto.ts`

### Deferred key pattern

The module holds an unresolved Promise at load time. Any call to `encryptText` or `decryptText` before `initCryptoKey()` is called will **await** — not fail — until the key is ready.

```typescript
let _resolveKey!: (key: CryptoKey) => void;
const _keyPromise = new Promise<CryptoKey>((resolve) => {
  _resolveKey = resolve;
});
```

This means `createEncryptedStorage()` and `atomWithStorage` are safe to instantiate at module load time, before the Provider mounts. Storage reads simply queue until the key arrives.

### `initCryptoKey(base64Key: string): Promise<void>`

Called once from `<CryptoKeyInit>` inside `JotaiProvider`. Imports the raw AES bytes as a non-extractable `CryptoKey` and resolves the deferred promise.

```
base64 string  (from Server Action, over HTTPS)
      │
      ▼  atob() → Uint8Array
crypto.subtle.importKey("raw", keyBytes, AES-GCM, non-extractable)
      │
      ▼
_resolveKey(key)  →  deferred promise resolves
                  →  all queued encrypt/decrypt calls proceed
```

### `encryptText(text: string): Promise<string | null>`

1. Awaits `_keyPromise`.
2. Generates a cryptographically random 12-byte IV via `crypto.getRandomValues`.
3. Encrypts UTF-8 text with AES-GCM.
4. Prepends IV to ciphertext and base64-encodes the combined buffer.

**Output format:** `base64( IV[12 bytes] || GCM ciphertext + auth tag )`

### `decryptText(encryptedBase64: string): Promise<string | null>`

1. Awaits `_keyPromise`.
2. Base64-decodes and slices: first 12 bytes = IV, remainder = ciphertext.
3. Decrypts with AES-GCM. Returns `null` on any failure — never throws.

> Minimum valid ciphertext: 28 bytes (12 IV + 16 GCM auth tag). Shorter inputs are rejected immediately.

---

## 4. Server Action — Key Derivation

**File:** `app/actions/crypto.action.ts`

```typescript
"use server";

export async function getClientEncryptionKey(scope = "global"): Promise<string>
```

### What it does

1. Reads `STORAGE_ENCRYPTION_KEY` from the server environment.
2. Derives an AES-GCM 256-bit key via PBKDF2 (100,000 iterations, SHA-256).
3. Exports the raw key bytes and returns them as a base64 string over HTTPS.

```
STORAGE_ENCRYPTION_KEY  (server env — never in browser bundle)
        │
        ▼  importKey("raw", ..., PBKDF2)
  keyMaterial
        │
        ▼  deriveKey(PBKDF2, salt="champions-arena:v1:{scope}", 100k, SHA-256)
  AES-GCM 256-bit CryptoKey  (extractable — server only)
        │
        ▼  exportKey("raw")  →  base64
  Returned to client over HTTPS
```

### Key stability

| Scenario | Outcome |
|---|---|
| Same `STORAGE_ENCRYPTION_KEY` + same `scope` | Identical key — existing `localStorage` data remains readable |
| `STORAGE_ENCRYPTION_KEY` rotated | New key — all existing `localStorage` data is unreadable; users re-authenticate |
| Different `scope` (e.g. different user ID) | Different key — per-user data isolation |

### `scope` parameter

| Value | Use case |
|---|---|
| `"global"` (default) | Deployment-wide stable key — simplest |
| User ID string | Per-user key — user A's data unreadable with user B's key |
| Session token hash | Per-session key — invalidated on logout without needing key rotation |

> Changing scope after data has been written makes existing `localStorage` unreadable for that scope. Clear `localStorage` on logout when using per-user scope.

---

## 5. Encrypted Storage Adapter

**File:** `lib/store/encrypted-storage.ts`

Implements a Jotai `AsyncStorage`-compatible adapter. Wraps `encryptText` and `decryptText`, which now transparently await the deferred key — no changes to the adapter's interface.

### SSR guard

On the server, a no-op async storage is returned. The atom hydrates from `initialState` server-side and updates from encrypted `localStorage` once the client key is initialized.

```typescript
if (typeof window === "undefined") {
  return createJSONStorage<T>(() => ({
    getItem: async () => null,
    setItem: async () => {},
    removeItem: async () => {},
  }));
}
```

---

## 6. Auth Atom

**File:** `lib/store/auth.atom.ts`

```typescript
const baseAuthAtom = atomWithStorage<AuthState>(
  "auth-state",
  initialState,
  createEncryptedStorage<AuthState>(),
  { getOnInit: true },
);

export const authAtom = unwrap(baseAuthAtom, (prev) => prev ?? initialState);
```

`getOnInit: true` triggers a storage read immediately on mount. The read calls `decryptText`, which awaits the deferred key promise. `unwrap()` returns `initialState` synchronously in the meantime — components never suspend. Once the key is ready, the read completes, the atom updates, and components re-render with restored auth state automatically.

---

## 7. Jotai Provider & Session Sync

**File:** `providers/jotai-provider.tsx`

### Component tree

```
<Provider>
  <CryptoKeyInit scope? />   ← calls Server Action → initCryptoKey()
  <SessionSync />            ← polls __ca_ux_i cookie
  {children}
</Provider>
```

### `<CryptoKeyInit scope?>`

```typescript
useEffect(() => {
  getClientEncryptionKey(scope)
    .then((base64Key) => initCryptoKey(base64Key))
    .catch(console.error);
}, [scope]);
```

Fires once on mount. Re-fires if `scope` changes (e.g. user ID becomes available after an initial anonymous load). On failure, logs the error; queued storage operations remain pending.

### `<SessionSync>`

Headless component bridging server cookies with client auth state.

| Trigger | Check | Action |
|---|---|---|
| Mount | `__ca_ux_i` present? | Clear atom if missing and currently authenticated |
| `window` focus | `__ca_ux_i` present? | Clear atom if missing and currently authenticated |
| Every 5 seconds | `__ca_ux_i` present? | Clear atom if missing and currently authenticated |

### `<JotaiProvider scope?>`

```tsx
<JotaiProvider scope={session?.user?.id}>
  {children}
</JotaiProvider>
```

Pass a stable user ID as `scope` (read server-side in the root layout) to derive a per-user encryption key.

---

## 8. Data Flow

### Initial page load

```
Provider mounts
  ├─ <CryptoKeyInit>: getClientEncryptionKey() ──────────────────────── [Server Action]
  └─ baseAuthAtom getOnInit: getItem() → decryptText() → awaits _keyPromise ─┐
                                                                              │ queued
Components render with initialState via unwrap()                              │
                                                                              │
Server Action resolves → initCryptoKey(base64) → _resolveKey() ──────────────┘
  └─ Queued decryptText() proceeds → baseAuthAtom updates
       └─ unwrap() propagates → components re-render with restored auth state
```

### Login

```
loginApi() → { user, token }
  └─ setAuth({ isAuthenticated: true, user })
       └─ setItem() → encryptText(JSON) → localStorage.setItem("auth-state", ciphertext)
```

### Server-side logout

```
Server clears __ca_ux_i cookie
  └─ SessionSync detects absence
       └─ setAuth({ isAuthenticated: false, user: null })
            └─ setItem() → overwrites localStorage with encrypted empty state
```

### Key rotation (breaking change — intentional)

```
STORAGE_ENCRYPTION_KEY rotated in deployment
  └─ New key derived by Server Action on next page load
       └─ Old localStorage ciphertext fails AES-GCM authentication
            └─ decryptText returns null → atomWithStorage falls back to initialState
                 └─ User sees logged-out state → re-authenticates cleanly
```

---

## 9. Security Model

### What this protects against

- **Casual `localStorage` inspection** — DevTools shows opaque base64, not tokens.
- **Physical device access** — data is useless ciphertext without the derived key.
- **JS bundle analysis for the master key** — `STORAGE_ENCRYPTION_KEY` is a server-only env var; only derived bytes travel to the client over HTTPS.
- **Cross-deployment data reuse** — rotating the master key invalidates all existing ciphertexts.

### Known limitations

| Threat | Required mitigation |
|---|---|
| XSS — attacker can invoke Server Action from the browser context | CSP headers, output sanitisation, `HttpOnly` session cookies |
| Network interception of derived key | HTTPS (enforced by Next.js in production) |
| Compromised server environment | Standard secret management (Vault, AWS Secrets Manager) |

> This is defence-in-depth for UI state persistence, not a replacement for server-side session management. Always pair with `HttpOnly` + `Secure` + `SameSite=Strict` session cookies.

---

## 10. Environment Variables

| Variable | Side | Required | Description |
|---|---|---|---|
| `STORAGE_ENCRYPTION_KEY` | **Server only** | Yes | Master secret for PBKDF2 derivation. No `NEXT_PUBLIC_` prefix — never bundled. |

```bash
# .env.local
STORAGE_ENCRYPTION_KEY=your-random-32-char-secret-here
```

Generate a secure value:

```bash
openssl rand -base64 32
```

> This variable is never sent to the browser. The client receives only the derived AES-GCM bytes, transmitted once per session over HTTPS via the Server Action.

---

## 11. Usage Examples

### Reading auth state

```tsx
"use client";
import { useAtom } from "jotai";
import { authAtom } from "@/lib/store/auth.atom";

export function UserAvatar() {
  const [auth] = useAtom(authAtom);
  if (!auth.isAuthenticated) return <GuestAvatar />;
  return <Avatar name={auth.user?.name} />;
}
```

### Login

```tsx
"use client";
import { useAtom } from "jotai";
import { authAtom } from "@/lib/store/auth.atom";

export function useLogin() {
  const [, setAuth] = useAtom(authAtom);

  const login = async (credentials: LoginCredentials) => {
    const { user, token } = await loginApi(credentials);
    setAuth({ isAuthenticated: true, user: { ...user, token } });
  };

  return { login };
}
```

### Logout with per-user scope cleanup

```tsx
const [, setAuth] = useAtom(authAtom);

const logout = async () => {
  setAuth({ isAuthenticated: false, user: null });
  await logoutApi(); // server clears __ca_ux_i cookie
  localStorage.removeItem("auth-state"); // wipe stale ciphertext for this scope
};
```

### Root layout with per-user scope

```tsx
// app/layout.tsx
import { getServerSession } from "next-auth";
import { JotaiProvider } from "@/providers/jotai-provider";

export default async function RootLayout({ children }) {
  const session = await getServerSession();

  return (
    <html>
      <body>
        <JotaiProvider scope={session?.user?.id}>
          {children}
        </JotaiProvider>
      </body>
    </html>
  );
}
```

### Route protection (middleware)

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const hasSession = request.cookies.has("__ca_ux_i");

  if (!hasSession && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }
}
```

---

## 12. Production Hardening Checklist

- [ ] Set `STORAGE_ENCRYPTION_KEY` to a randomly generated 32-byte value — server-only, no `NEXT_PUBLIC_` prefix.
- [ ] Verify it is absent from the client bundle: inspect `.next/static/` after `next build`.
- [ ] Rotate `STORAGE_ENCRYPTION_KEY` on suspected breach — all `localStorage` data is invalidated; users re-authenticate.
- [ ] Pass user ID as `scope` to `<JotaiProvider>` for per-user key isolation.
- [ ] Call `localStorage.removeItem("auth-state")` on logout when using per-user scope.
- [ ] Add Content Security Policy headers in `next.config.js` — XSS is the primary attack vector.
- [ ] Pair with `HttpOnly` + `Secure` + `SameSite=Strict` session cookies for actual session management.
- [ ] Monitor `getClientEncryptionKey` Server Action for errors in your observability platform.
- [ ] Move the PBKDF2 salt in `crypto.action.ts` to an environment variable for true environment isolation between staging and production.
- [ ] For high-security requirements, derive per-session keys (pass a session token hash as `scope`) so re-login always produces a new key, even without rotating the master secret.
