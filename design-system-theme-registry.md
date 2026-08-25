# Vanilla JS Design System: Theme Registry Architecture
## A Framework-Free Approach to Scalable, Themeable Web Components

> *This design system documents the architecture patterns used in [improved-parakeet](https://github.com/HeatherB/improved-parakeet), demonstrating how modern browser APIs can replace traditional CSS-in-JS frameworks while maintaining design consistency and developer ergonomics.*

---

## 🎯 Design Goals

### The Challenge
Build a themeable, component-based portfolio site without:
- Build tools or bundlers (zero-config deployment to GitHub Pages)
- CSS-in-JS runtime overhead
- Framework lock-in (React, Vue, etc.)
- Global CSS conflicts

### Constraints & Requirements
| Requirement | Solution |
|-------------|----------|
| Theme switching without page reload | Constructable Stylesheets + adoptedStyleSheets |
| Style encapsulation per component | Shadow DOM |
| Cross-component communication | Custom Events with `composed: true` |
| User preference persistence | localStorage |
| Zero build step | Native ES6 modules, no transpilation |

---

## 🏗️ Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                      DESIGN SYSTEM LAYERS                        │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    DESIGN TOKENS                             ││
│  │         CSS Custom Properties (:root variables)              ││
│  │     Colors, typography, spacing, shadows, etc.               ││
│  └─────────────────────────────────────────────────────────────┘│
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    RESET / NORMALIZE                         ││
│  │           Shared base styles (CSSStyleSheet)                 ││
│  │        Applied to ALL component Shadow DOMs                  ││
│  └─────────────────────────────────────────────────────────────┘│
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    THEME REGISTRY                            ││
│  │       window.themeRegistry = { downtown, retro, ... }        ││
│  │         Each theme = CSSStyleSheet instance                  ││
│  └─────────────────────────────────────────────────────────────┘│
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    WEB COMPONENTS                            ││
│  │        <home-content>, <about-content>, etc.                 ││
│  │     shadowRoot.adoptedStyleSheets = [reset, theme]           ││
│  └─────────────────────────────────────────────────────────────┘│
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                 EVENT-DRIVEN THEME SWITCHING                 ││
│  │           CustomEvent('theme-change', { composed: true })    ││
│  │              Components listen & re-apply sheets             ││
│  └─────────────────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────────────────┘
```

---

## 📐 Design Tokens

All design tokens are defined as CSS Custom Properties on `:root`, providing a single source of truth:

```css
:root {
  /* Color Palette */
  --theme-primary-color: #f4824e;    /* Brand orange */
  --theme-secondary-color: #0DDFFF;  /* Accent teal */
  --theme-background-color: #E5E4E2; /* Neutral background */
  --theme-text-color: #000000;
  --theme-color-black: #000000;
  --theme-color-white: #ffffff;
  
  /* Component-Specific */
  --theme-footer-bg: #000000;
  --theme-footer-color: #ffffff;
  --theme-intro-bg: #ffffff;
  --theme-shadow-color: #ffffff;
  
  /* Typography Scale */
  --theme-font-family: playfair-display, serif;
  --theme-font-decorative: giddyup-std, sans-serif;
  --theme-font-subhead: prestige-elite-std, monospace;
  --theme-font-sans: "Segoe UI", Frutiger, "Frutiger Linotype", 
                     "Dejavu Sans", "Helvetica Neue", Arial, sans-serif;
}
```

### Token Naming Convention
```
--theme-{category}-{property}[-{variant}]

Examples:
--theme-primary-color       (category: primary, property: color)
--theme-font-family         (category: font, property: family)
--theme-footer-bg           (category: footer, property: bg)
```

---

## 🎨 Theme Registry Pattern

### Core Implementation

The theme registry uses the **Constructable Stylesheets** API to create reusable, adoptable stylesheet objects:

```javascript
// theme-registry.js
window.themeRegistry = window.themeRegistry || {};

window.registerTheme = function(name, cssText) {
  const sheet = new CSSStyleSheet();
  sheet.replaceSync(cssText);        // Parse CSS synchronously
  window.themeRegistry[name] = sheet; // Store for later adoption
};

window.getThemeSheet = function(name) {
  return window.themeRegistry[name] || window.themeRegistry.downtown;
};
```

### Why Constructable Stylesheets?

| Traditional Approach | Constructable Stylesheets |
|---------------------|---------------------------|
| `<style>` tags duplicated per component | Single CSSStyleSheet shared across Shadow DOMs |
| Re-parsing CSS on every component mount | Parse once, adopt many times |
| Theme change = DOM manipulation | Theme change = swap sheet reference |
| Memory: O(n) where n = components | Memory: O(1) per theme |

### Registering a Theme

Each theme is a self-registering module:

```javascript
// home-downtown.js
window.registerTheme('downtown', `
  #content_wrapper {
    &.home {
      background: url(./assets/images/bg/boat.jpg) 50% 75% no-repeat;
      background-size: cover;
      
      h1 {
        font-size: 85px;
        max-width: 380px;
        text-shadow: 1px 1px 1px var(--theme-shadow-color);
        
        .hey {
          color: var(--theme-secondary-color);
          font-family: var(--theme-font-decorative);
        }
        .proper {
          color: var(--theme-primary-color);
          font-size: 63px;
        }
        .detail {
          font-family: var(--theme-font-subhead);
          font-size: 21px;
        }
      }
    }
  }
  
  @media (min-width: 1024px) {
    #content_wrapper {
      &.home {
        section {
          width: 40vw;
          transform: translateX(-110%);
        }
      }
    }
  }
`);
```

---

## 🧩 Component Integration

### Adopting Stylesheets in Shadow DOM

Components adopt multiple stylesheets (reset + theme) via the `adoptedStyleSheets` API:

```javascript
class HomeContent extends HTMLElement {
  constructor() {
    super();
    const shadow = this.attachShadow({ mode: "open" });
    shadow.appendChild(template.content.cloneNode(true));
    
    // Apply initial theme from user preference
    const initialTheme = localStorage.getItem('theme') || 'downtown';
    this.applyTheme(initialTheme);
    
    // Listen for theme changes
    document.addEventListener('theme-change', (e) => {
      this.applyTheme(e.detail.theme);
    });
  }
  
  applyTheme(theme) {
    // Swap stylesheets without DOM manipulation
    this.shadowRoot.adoptedStyleSheets = [
      window.resetSheet,          // Shared reset/normalize
      window.getThemeSheet(theme) // Theme-specific styles
    ];
    
    // Update class for CSS hooks
    const wrapper = this.shadowRoot.getElementById('content_wrapper');
    wrapper.className = `home ${theme}`;
  }
}
```

### Stylesheet Layering

```
┌─────────────────────────────────────────┐
│          Component Shadow DOM           │
│  ┌───────────────────────────────────┐  │
│  │  adoptedStyleSheets[0]: resetSheet │  │  ← Base reset/normalize
│  ├───────────────────────────────────┤  │
│  │  adoptedStyleSheets[1]: themeSheet │  │  ← Theme overrides
│  ├───────────────────────────────────┤  │
│  │  <style> inline (if needed)        │  │  ← Component-specific
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

---

## 🔄 Event-Driven Theme Switching

### Cross-Shadow DOM Communication

The theme switcher dispatches a `CustomEvent` that crosses Shadow DOM boundaries:

```javascript
// theme-switcher.js
class ThemeSwitcher extends HTMLElement {
  constructor() {
    super();
    const shadow = this.attachShadow({ mode: "open" });
    
    shadow.querySelector('.switch-theme')?.addEventListener('change', (event) => {
      const themeEvent = new CustomEvent('theme-change', {
        bubbles: true,    // Bubbles up through ancestors
        composed: true,   // Crosses Shadow DOM boundaries
        detail: {
          theme: event.target.value
        }
      });
      
      this.dispatchEvent(themeEvent);
    });
  }
}
```

### Event Flow

```
User selects theme
        │
        ▼
┌───────────────────┐
│ <theme-switcher>  │  dispatchEvent(CustomEvent)
│   (Shadow DOM)    │         │
└───────────────────┘         │
        │                     │ composed: true
        │                     │ (crosses shadow boundary)
        ▼                     ▼
┌───────────────────┐    ┌───────────────────┐
│ document listener │───▶│ <home-content>    │
└───────────────────┘    │   applyTheme()    │
        │                └───────────────────┘
        │                     │
        ▼                     ▼
┌───────────────────┐    ┌───────────────────┐
│ localStorage.set  │    │ adoptedStyleSheets│
│ ('theme', value)  │    │ = [reset, theme]  │
└───────────────────┘    └───────────────────┘
```

---

## 🔧 Reset/Normalize Layer

A shared reset stylesheet ensures consistent baseline styles across all components:

```javascript
// reset-styles.js
const resetCSS = `
  *, *::before, *::after {
    box-sizing: border-box;
  }
  
  body, h1, h2, h3, h4, p, figure, blockquote, dl, dd {
    margin-block-end: 0;
  }
  
  h1, h2, h3, h4, button, input, label {
    line-height: 1.1;
  }
  
  h1, h2, h3, h4 {
    text-wrap: balance;
  }
  
  a:not([class]) {
    text-decoration-skip-ink: auto;
    color: currentColor;
  }
  
  img, picture {
    max-width: 100%;
    display: block;
  }
`;

const resetSheet = new CSSStyleSheet();
resetSheet.replaceSync(resetCSS);
window.resetSheet = resetSheet;
```

---

## 📝 Adding a New Theme

### Step 1: Create Theme File

```javascript
// home-mytheme.js
window.registerTheme('mytheme', `
  #content_wrapper {
    &.home {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      
      h1 {
        color: #ffffff;
        text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        
        .hey { color: #ffd700; }
        .proper { color: #ffffff; }
        .detail { color: #e0e0e0; }
      }
      
      p {
        color: #ffffff;
        background: rgba(0,0,0,0.2);
        padding: 1.5rem;
        border-radius: 8px;
      }
    }
  }
`);
```

### Step 2: Include Script

```html
<script src="assets/styles/themes/home-mytheme.js"></script>
```

### Step 3: Add to Theme Switcher

```html
<option value="mytheme">My Theme</option>
```

### Step 4: (Optional) Add Body-Level Styles

```css
/* themes.css */
body.mytheme {
  background: #667eea;
}

body.mytheme .container {
  /* Theme-specific layout adjustments */
}
```

---

## ✅ Design System Checklist

When adding new components or themes, verify:

- [ ] Component uses Shadow DOM (`attachShadow({ mode: "open" })`)
- [ ] Component adopts `window.resetSheet` as first stylesheet
- [ ] Component listens for `theme-change` event
- [ ] Theme uses CSS Custom Properties from `:root`
- [ ] Theme is self-registering via `window.registerTheme()`
- [ ] Responsive breakpoints are consistent (1024px desktop threshold)
- [ ] Accessibility: focus-visible styles inherited from global

---

## 🎯 Benefits of This Architecture

| Benefit | How It's Achieved |
|---------|-------------------|
| **Zero build step** | Native browser APIs, no transpilation |
| **Instant theme switching** | Swap CSSStyleSheet references, no DOM updates |
| **Memory efficient** | Single stylesheet instance shared across components |
| **Style encapsulation** | Shadow DOM prevents leakage |
| **Maintainable** | Each theme is a standalone file |
| **Extensible** | Add themes without touching existing code |
| **Persistent preferences** | localStorage integration |
| **Framework agnostic** | Pure Web Components, works anywhere |

---

## 🔗 Related Resources

- [Constructable Stylesheets (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleSheet/CSSStyleSheet)
- [adoptedStyleSheets (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Document/adoptedStyleSheets)
- [Web Components (MDN)](https://developer.mozilla.org/en-US/docs/Web/Web_Components)
- [Custom Events (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent)

---

*This design system demonstrates that modern browser APIs can replace complex build toolchains while maintaining—or improving—developer experience and runtime performance.*
