# Image Configuration Documentation

## Overview

The `imageConfig` object provides a centralized configuration system for handling image paths in Next.js applications. It dynamically constructs image URLs based on environment variables and runtime context, supporting both local and remote image sources.

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

## Usage Examples

### Basic Image URL Construction
```javascript
import { imageConfig } from './path/to/imageConfig';

const logoUrl = imageConfig.url('logo.png');
const heroImageUrl = imageConfig.url('assets/hero.jpg');

// Results based on environment:
// - Local: '/logo.png', '/assets/hero.jpg'
// - With CDN: 'https://cdn.example.com/logo.png', 'https://cdn.example.com/assets/hero.jpg'
```

### Next.js Image Component Integration
```javascript
import Image from 'next/image';
import { imageConfig } from './path/to/imageConfig';

// Basic usage
<Image 
  src={imageConfig.url('profile.jpg')} 
  alt="Profile" 
  width={200} 
  height={200} 
/>

// With remote/local detection
const { remote, local } = imageConfig.nextImageSrc;
<Image 
  src={imageConfig.url('banner.jpg')} 
  alt="Banner" 
  width={800} 
  height={400}
  priority={remote ? true : false}
/>
```

### Environment Configuration
```bash
# Local development
NEXT_PUBLIC_IMAGE_PATH=/assets

# Production with CDN
NEXT_PUBLIC_IMAGE_PATH=https://cdn.example.com

# Production with subdirectory
NEXT_PUBLIC_IMAGE_PATH=/my-app/images
```

### Dynamic Image Paths
```javascript
// Works with relative paths
imageConfig.url('icons/star.svg')        // '/icons/star.svg' or 'https://cdn.example.com/icons/star.svg'

// Works with absolute paths
imageConfig.url('/images/hero.jpg')      // '/images/hero.jpg' or 'https://cdn.example.com/images/hero.jpg'

// Passes through complete URLs
imageConfig.url('https://external.com/image.png') // 'https://external.com/image.png'
```

## Configuration Details

### Properties

#### `basePath` (getter)
Returns the base path for images, determined by priority:
1. `NEXT_PUBLIC_IMAGE_PATH` environment variable (with trailing slash removed)
2. Empty string if running in browser environment
3. `DEFAULT_PATH` ('/') as fallback for server-side rendering

#### `url(path: string)`
Constructs complete image URLs by combining the base path with the provided path.

**Parameters:**
- `path` - The image path (relative or absolute URL)

**Returns:** Complete URL string

#### `nextImageSrc` (getter)
Provides Next.js Image component compatible source configuration.

**Returns:** Object with `remote` and `local` properties

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
