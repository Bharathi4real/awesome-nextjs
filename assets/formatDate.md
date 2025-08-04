# `formatReadableDate`

Converts an ISO 8601 UTC date string (e.g. `2025-07-19T05:35:59.041000Z`) into a human-readable date format using `Intl.DateTimeFormat`.

- Automatically formats based on your system or provided locale
- Customizable with `Intl.DateTimeFormatOptions`
- Safe fallback for invalid dates
- Useful for rendering readable timestamps in UIs like posts, logs, notifications, and more

---

## Function Definition

```ts
/**
 * Converts ISO date string to a readable format
 *
 * @param isoDate - A valid ISO 8601 date string (e.g. "2025-07-19T05:35:59.041000Z")
 * @param options - Optional Intl.DateTimeFormatOptions to customize output
 * @param locale - Optional BCP 47 locale string (defaults to 'en-US')
 * @returns Formatted date string or original input if invalid
 *
 * @example
 * formatReadableDate('2025-07-19T05:35:59.041000Z');
 * // => "19 Jul 2025, 6:35 AM" (based on en-US locale and browser timezone)
 *
 * @example
 * formatReadableDate('2025-07-19T05:35:59.041000Z', { weekday: 'long' });
 * // => "Saturday, 19 Jul 2025, 06:35"
 *
 * @example
 * formatReadableDate('2025-07-19T05:35:59.041000Z', { hour12: false });
 * // => "19 Jul 2025, 06:35"
 *
 * @example
 * formatReadableDate('2025-07-19T05:35:59.041000Z', undefined, 'en-US');
 * // => "Jul 19, 2025, 1:35 AM"
 *
 * @example
 * formatReadableDate('invalid-date');
 * // => "invalid-date"
 */
export function formatReadableDate(
  isoDate: string,
  options?: Intl.DateTimeFormatOptions,
  locale: string = 'en-US'
): string {
  try {
    const date = new Date(isoDate);
     if (Number.isNaN(date.getTime())) {
      throw new Error('Invalid date');
    }
    return new Intl.DateTimeFormat(locale, {
      year: 'numeric',
      month: 'short',
      day: '2-digit',
      hour: 'numeric',
      minute: '2-digit',
      hour12: true,
      ...options,
    }).format(date);
  } catch (error) {
    console.error('Date formatting error:', error);
    return isoDate;
  }
}
```

## Usage

### Import
```ts
import { formatReadableDate } from '@/lib/utils/dateFormat';
```

### Examples

| Call | Output (Local Timezone) |
|------|--------------------------|
| `formatReadableDate('2025-07-19T05:35:59.041000Z')` | "Jul 19, 2025, 11:05 AM" |
| `formatReadableDate('2025-07-19T05:35:59.041000Z', { weekday: 'long' })` | "Saturday, Jul 19, 2025, 11:05 AM" |
| `formatReadableDate('invalid-date')` | "invalid-date" |

## 🔧 Parameters

| Name | Type | Description |
|------|------|-------------|
| `isoDate` | `string` | A valid ISO 8601 date string (UTC or local ISO) |
| `options` | `Intl.DateTimeFormatOptions?` | Optional object to customize output (see [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat/DateTimeFormat)) |
| `locale` | `string` (default: `'en-US'`) | Optional BCP 47 language tag (e.g. `'en-GB'`, `'fr-FR'`) |

## Returns

**`string`** — A locale-formatted readable date string.

If the input is invalid, it logs an error and returns the original string.

## Example in a Component

```tsx
import { formatReadableDate } from '@/lib/utils/dateFormat';

export function PostMeta({ publishedAt }: { publishedAt: string }) {
  return (
    <time
      dateTime={publishedAt}
      className="text-sm text-muted-foreground"
    >
      {formatReadableDate(publishedAt)}
    </time>
  );
}
```

This will render:
🗓️ "Jul 19, 2025, 11:05 AM" or similar in your timezone.

## Tip

Use this utility for:
- Blog or article publish dates
- Audit logs and timestamps
- Comments and activity feeds
- Transaction or notification times

Supports custom formatting and localization using browser-native `Intl`.

## Contribution

Want to improve locale support or add timezone presets? Open a pull request!
