# Image Configuration Documentation

## Overview

The `imageConfig` object provides a centralized configuration system for handling image paths in Next.js applications. It dynamically constructs image URLs based on environment variables and runtime context, supporting both local and remote image sources.

## Configuration Object

### Properties

#### `basePath` (getter)
Returns the base path for images, determined by the following priority:
1. `NEXT_PUBLIC_IMAGE_PATH` environment variable (with trailing slash removed)
2. Empty string if running in browser environment
3. `DEFAULT_PATH` ('/') as fallback for server-side rendering

```javascript
// Examples of basePath values:
// - '/images' (from env var)
// - 'https://cdn.example.com' (from env var)
// - '' (browser context)
// - '/' (SSR fallback)
```

#### `url(path: string)`
Constructs complete image URLs by combining the base path with the provided path.

**Parameters:**
- `path` - The image path (relative or absolute URL)

**Returns:** Complete URL string

**Behavior:**
- Returns the path unchanged if it's already a complete URL (starts with `http://` or `https://`)
- Removes leading slash from path if present
- Formats base path to ensure proper URL structure
- Combines base path and image path with appropriate separators

```javascript
// Usage examples:
imageConfig.url('logo.png')           // '/logo.png' or 'https://cdn.example.com/logo.png'
imageConfig.url('/assets/hero.jpg')   // '/assets/hero.jpg' or 'https://cdn.example.com/assets/hero.jpg'
imageConfig.url('https://external.com/image.png') // 'https://external.com/image.png'
```

#### `nextImageSrc` (getter)
Provides Next.js Image component compatible source configuration.

**Returns:** Object with `remote` and `local` properties
- `remote`: Base path if it's a remote URL (starts with http/https), otherwise `undefined`
- `local`: Base path without leading slash if it's local, otherwise `undefined`

```javascript
// Examples:
// For basePath = 'https://cdn.example.com':
// { remote: 'https://cdn.example.com', local: undefined }

// For basePath = '/images':
// { remote: undefined, local: 'images' }
```

## Usage Patterns

### Basic Image URL Construction
```javascript
import { imageConfig } from './path/to/imageConfig';

const logoUrl = imageConfig.url('logo.png');
const heroImageUrl = imageConfig.url('assets/hero.jpg');
```

### Next.js Image Component Integration
```javascript
import Image from 'next/image';
import { imageConfig } from './path/to/imageConfig';

const { remote, local } = imageConfig.nextImageSrc;

// For remote images
if (remote) {
  <Image src={imageConfig.url('image.jpg')} alt="Description" width={500} height={300} />
}

// For local images
if (local) {
  <Image src={imageConfig.url('image.jpg')} alt="Description" width={500} height={300} />
}
```

## Environment Configuration

Set the `NEXT_PUBLIC_IMAGE_PATH` environment variable to customize the base path:

```bash
# Local development
NEXT_PUBLIC_IMAGE_PATH=/assets

# Production with CDN
NEXT_PUBLIC_IMAGE_PATH=https://cdn.example.com

# Production with subdirectory
NEXT_PUBLIC_IMAGE_PATH=/my-app/images
```

## Full Source Code

```typescript
const DEFAULT_PATH = '/';

export const imageConfig = {
  get basePath() {
    return (
      process.env.NEXT_PUBLIC_IMAGE_PATH?.replace(/\/$/, '') ||
      (typeof window !== 'undefined' ? '' : DEFAULT_PATH)
    );
  },
  url(path: string) {
    if (path.startsWith('http://') || path.startsWith('https://')) return path;
    const cleanPath = path.startsWith('/') ? path.slice(1) : path;
    const base = this.basePath || '';
    const formattedBase = base.startsWith('http')
      ? base
      : base.startsWith('/')
        ? base
        : `/${base}`;
    return `${formattedBase}${formattedBase.endsWith('/') ? '' : '/'}${cleanPath}`;
  },
  get nextImageSrc() {
    const isRemote =
      this.basePath.startsWith('http://') ||
      this.basePath.startsWith('https://');
    return {
      remote: isRemote ? this.basePath : undefined,
      local: !isRemote ? this.basePath.replace(/^\//, '') : undefined,
    };
  },
};
```

## Key Features

- **Environment-aware**: Automatically adapts based on runtime environment (browser vs server)
- **Flexible path handling**: Supports both local and remote image sources
- **URL safety**: Prevents double slashes and ensures proper URL formatting
- **Next.js compatible**: Provides helper for Next.js Image component configuration
- **Absolute URL passthrough**: Leaves complete URLs unchanged

## Edge Cases Handled

- Trailing slashes in environment variables are automatically removed
- Leading slashes in image paths are normalized
- Complete URLs (http/https) bypass path manipulation
- Server-side rendering fallback when window is undefined
- Empty base paths are handled gracefully
