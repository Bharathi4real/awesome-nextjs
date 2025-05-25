# Custom Tailwind Utility: `.container`

This file defines a responsive `.container` utility class using Tailwind CSS v4’s new `@layer` and `@variant` features. It ensures consistent layout padding and max-width constraints across breakpoints.

## Features

- Centers content with horizontal margins
- Adds consistent horizontal padding
- Sets maximum widths at various breakpoints
- Fully responsive using Tailwind's `@variant` syntax

---

```css
@layer utilities {
  .container {
    margin-inline: auto;
    box-sizing: border-box;
    padding-inline: 10px;
  }

  @variant lg {
    .container {
      max-width: 960px;
    }
  }

  @variant xl {
    .container {
      max-width: 1200px;
    }
  }

  @variant sm {
    .container {
      padding: 10px;
    }
  }
}
```

---

## Contribution
Want to suggest axios config options? Open a pull request!
