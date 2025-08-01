# `formatPathSegment`

Formats a specific segment from a pathname into a human-readable string by:

- Extracting a segment by index (supports negative index for "last")
- Splitting the segment on `-`
- Capitalizing each word
- Joining with spaces

Useful for generating dynamic page titles, breadcrumbs, or headings from URL paths.

---

## Function Definition

```ts
/**
 * Get and format a specific segment from a pathname.
 *
 * @param pathname - Full pathname (e.g. /industrial/high-density-storage)
 * @param index - Segment index (0-based). Use -1 for last segment.
 * @returns Capitalized words (e.g. "High Density Storage")
 */
export function formatPathSegment(pathname: string, index: number): string {
  const segments = pathname.split('/').filter(Boolean);

  const segment = index === -1 ? segments[segments.length - 1] : segments[index] || '';

  if (!segment) return '';

  return segment
    .split('-')
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
    .join(' ');
}
```

---

## Usage

### Import

```ts
import { formatPathSegment } from '@/lib/utils/formatPathSegment';
```

### Example Path

```ts
const pathname = '/products/electrical-components/high-density-storage';
```

### Examples

| Call                                              | Output                    |
|---------------------------------------------------|---------------------------|
| `formatPathSegment(pathname, 0)`                  | `"Products"`              |
| `formatPathSegment(pathname, 1)`                  | `"Electrical Components"` |
| `formatPathSegment(pathname, -1)`                 | `"High Density Storage"`  |
| `formatPathSegment('/blog/my-post', -1)`          | `"My Post"`               |
| `formatPathSegment('/only-one', 0)`               | `"Only One"`              |
| `formatPathSegment('/', 0)`                       | `""`                      |

---

## 🔧 Parameters

| Name       | Type     | Description                                                                 |
|------------|----------|-----------------------------------------------------------------------------|
| `pathname` | `string` | Full pathname (from `usePathname()` or router)                              |
| `index`    | `number` | Index of the segment to extract (0-based). Use `-1` for the last segment.   |

---

## Returns

**`string`** — A formatted version of the requested segment in title case. Returns an empty string if the index is out of bounds.

---

## Example in a Component

Here’s a generic example of how to use `formatPathSegment` in a React component with the `next/navigation` hook:

```tsx
'use client';

import { usePathname } from 'next/navigation';
import { formatPathSegment } from '@/lib/utils/formatPathSegment';

export function PageHeader() {
  const pathname = usePathname();
  const sectionTitle = formatPathSegment(pathname, -1); // Use last segment as header

  return (
    <header className="py-12 text-center">
      <h1 className="text-4xl font-bold tracking-tight text-gray-900 dark:text-white">
        {sectionTitle}
      </h1>
      <p className="mt-2 text-lg text-gray-600 dark:text-gray-300">
        Welcome to the {sectionTitle} section
      </p>
    </header>
  );
}
```

This will render a readable title like `"High Density Storage"` when the URL is `/products/electrical-components/high-density-storage`.

---

## Tip

Use this utility for:

- Dynamic `<title>` and `<meta>` tags
- Breadcrumbs
- Hero/banner headers
- Page intros

Supports optional segments and will safely return `""` if the index is invalid or segment is missing.

## Contribution
Want to suggest axios config options? Open a pull request!
