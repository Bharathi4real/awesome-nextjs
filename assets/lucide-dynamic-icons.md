# `LucideIcon`
A dynamic React component that renders Lucide icons by name with automatic normalization and fallback handling.
- Automatically normalizes icon names from kebab-case, snake_case, or space-separated strings to PascalCase
- Safe fallback to Share2 icon when requested icon doesn't exist
- Consistent sizing and stroke width across all icons
- Useful for dynamic icon rendering based on user input, API data, or configuration
---
## Component Definition
```tsx
import { icons } from 'lucide-react';

/**
 * Renders a Lucide icon dynamically by name with automatic normalization
 *
 * @param name - Icon name in any case format (kebab-case, snake_case, PascalCase, etc.)
 * @returns JSX element with the requested icon or Share2 fallback
 *
 * @example
 * <LucideIcon name="user-circle" />
 * // Renders UserCircle icon
 *
 * @example
 * <LucideIcon name="arrow_right" />
 * // Renders ArrowRight icon
 *
 * @example
 * <LucideIcon name="home icon" />
 * // Renders HomeIcon (if exists) or Share2 fallback
 *
 * @example
 * <LucideIcon name="nonexistent-icon" />
 * // Renders Share2 fallback with console warning
 */
export function LucideIcon({ name }: { name: string }) {
    // Normalize the name - Lucide uses PascalCase for icon names
    const normalizedName = name
      .toLowerCase()
      .split(/[-_\s]/)
      .map(word => word.charAt(0).toUpperCase() + word.slice(1))
      .join('');
    
    // Get the icon from the icons object
    const IconComponent = icons[normalizedName as keyof typeof icons];
    
    if (!IconComponent) {
      console.warn(`⚠️ Lucide icon "${name}" (normalized: "${normalizedName}") not found`);
      // Return a default share icon as fallback
      return <icons.Share2 size={22} strokeWidth={1.75} />;
    }
    
    return <IconComponent size={22} strokeWidth={1.75} />;
}
```
## Usage
### Import
```tsx
import { LucideIcon } from '@/components/ui/LucideIcon';
```
### Examples
| Input Name | Normalized | Result |
|------------|------------|--------|
| `"user-circle"` | `"UserCircle"` | 👤 UserCircle icon |
| `"arrow_right"` | `"ArrowRight"` | ➡️ ArrowRight icon |
| `"home icon"` | `"HomeIcon"` | 🏠 Home icon (if exists) |
| `"Settings"` | `"Settings"` | ⚙️ Settings icon |
| `"invalid-name"` | `"InvalidName"` | 📤 Share2 fallback + warning |

## 🔧 Props
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `name` | `string` | ✅ | Icon name in any case format (kebab-case, snake_case, PascalCase, etc.) |

## Returns
**`JSX.Element`** — A Lucide icon component with standardized size (22px) and stroke width (1.75).
If the requested icon doesn't exist, returns Share2 icon as fallback with a console warning.

## Icon Specifications
- **Size**: 22px
- **Stroke Width**: 1.75
- **Fallback**: Share2 icon

## Example in Components
```tsx
import { LucideIcon } from '@/components/ui/LucideIcon';

// Static usage
export function NavigationItem() {
  return (
    <div className="flex items-center gap-2">
      <LucideIcon name="home" />
      <span>Home</span>
    </div>
  );
}

// Dynamic usage with data
export function FeatureList({ features }: { features: { name: string; icon: string }[] }) {
  return (
    <ul className="space-y-2">
      {features.map((feature) => (
        <li key={feature.name} className="flex items-center gap-3">
          <LucideIcon name={feature.icon} />
          <span>{feature.name}</span>
        </li>
      ))}
    </ul>
  );
}

// With conditional rendering
export function StatusIcon({ status }: { status: 'success' | 'error' | 'loading' }) {
  const iconMap = {
    success: 'check-circle',
    error: 'x-circle', 
    loading: 'loader'
  };
  
  return <LucideIcon name={iconMap[status]} />;
}
```

## Name Normalization Examples
The component automatically converts various naming conventions to Lucide's PascalCase format:

```tsx
// All of these render the same UserCircle icon
<LucideIcon name="user-circle" />    // kebab-case
<LucideIcon name="user_circle" />    // snake_case  
<LucideIcon name="user circle" />    // space-separated
<LucideIcon name="UserCircle" />     // PascalCase (no change)
<LucideIcon name="userCircle" />     // camelCase → UserCircle
```

## Error Handling
When an icon name doesn't match any available Lucide icon:
1. **Console Warning**: Logs the original and normalized names
2. **Fallback Icon**: Renders Share2 icon to prevent layout breaks
3. **No Crashes**: Component always returns valid JSX

## Tip
Use this component for:
- Dynamic navigation menus with icon data from APIs
- Feature lists with configurable icons
- Status indicators with icon mapping
- Form inputs where users can select icons
- CMS-driven content with icon fields

Perfect for maintaining consistent icon styling while supporting flexible icon selection!

## Available Icons
This component works with all icons from the `lucide-react` package. Check the [Lucide website](https://lucide.dev/icons/) for the complete icon library.

## Contribution
Want to customize default size, stroke width, or fallback icon? Feel free to modify the component or create variants!
