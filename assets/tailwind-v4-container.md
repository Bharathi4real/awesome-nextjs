# Custom Tailwind Utility: `.container`

This file defines a responsive .container utility class using Tailwind CSS v4’s @layer system. It ensures consistent layout width, padding, and responsiveness across all screen sizes—from small mobile devices to large desktops.

## Features

- Horizontally centers content using `margin-inline: auto;`
- Applies smart horizontal padding via `padding-inline`
- Sets logical `max-width` values for various screen widths
- Supports ultra-small mobile breakpoints (`320px`, `370px`, `420px`)
-  Scales up nicely to tablets, laptops, and wide desktop displays
---

```css
@layer utilities {
  .container {
    margin-inline: auto;
    width: 100%;
    padding-inline: 1rem; /* 16px horizontal padding */
    box-sizing: border-box;
  }

  @media (min-width: 320px) {
    .container {
      max-width: 300px; /* Tiny phones */
    }
  }

  @media (min-width: 375px) {
    .container {
      max-width: 350px; /* Medium phones */
    }
  }

  @media (min-width: 425px) {
    .container {
      max-width: 400px; /* Large phones */
    }
  }

  @media (min-width: 640px) {
    .container {
      max-width: 600px; /* Small tablets */
    }
  }

  @media (min-width: 768px) {
    .container {
      max-width: 720px; /* Tablets */
    }
  }

  @media (min-width: 1024px) {
    .container {
      max-width: 960px; /* Small Laptops */
    }
  }

  @media (min-width: 1280px) {
    .container {
      max-width: 1200px; /* Large Laptops */
    }
  }

  @media (min-width: 1536px) {
    .container {
      max-width: 1440px; /* Large screens */
    }
  }

  @media (min-width: 1920px) {
    .container {
      max-width: 1720px; /* Ultra-wide displays */
    }
  }
}
```
---

## Usage Example

```html
<div class="container">
  <p>This content will be neatly padded and centered at all breakpoints.</p>
</div>
```

---

## Contribution
Want to suggest axios config options? Open a pull request!
