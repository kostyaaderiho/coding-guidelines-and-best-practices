# Assets

### Overview

This page documents best practices and standards for managing static assets (images, SVGs, icons, graphics) in UI.

### Best Practices

#### SVG optimization

If you are working with SVG format, always optimize it by https://svgomg.net/ before adding it to the project.

This removes unnecessary metadata, comments, and reduces file size by 50-80% on average.

**Optimization checklist:**

1. Remove comments and metadata
2. Simplify paths (reduce precision to 1-2 decimal places)
3. Remove unused IDs and classes
4. Minify colors
5. Convert shapes to paths where beneficial

Incorrect

```
<?xml version="1.0" encoding="UTF-8"?>
<!-- Generator: Adobe Illustrator 25.0.0, SVG Export Plug-In -->
<svg version="1.1" xmlns="http://www.w3.org/2000/svg"
     x="0px" y="0px" viewBox="0 0 100 100"
     style="enable-background:new 0 0 100 100;"
     xml:space="preserve">
  <g id="Layer_1">
    <circle fill="#FF0000" cx="50.000000" cy="50.000000" r="40.000000"/>
  </g>
</svg>
```

Correct

```typescript
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
    <circle fill="#f00" cx="50" cy="50" r="40"/>
</svg>
```

#### Choose the Right Format

- **SVG** ΓÇô icons, logos, and simple graphics ΓÇö scalable and lightweight.
- **WebP or AVIF** ΓÇô photos and complex images ΓÇö smaller file sizes and better quality than JPEG/PNG.
- **PNG** ΓÇô for images requiring transparency.
- **JPEG** ΓÇô photographs with no transparency needs.
- **GIF** ΓÇô Γ¥î Avoid due to large files, limited colors. Use video instead

#### File Naming Conventions

Use consistent, descriptive names for all assets followed by the `camelCase` pattern.

Incorrect

```
UserAvatar.svg
```

Correct

```
userAvatar.svg
```

#### Use Descriptive 'alt' text

Provide meaningful `alt` attributes for accessibility and SEO

Incorrect

```html
<img src="photo.jpg" alt="photo" /> <img src="user-avatar.png" alt="image" />
```

Correct

```html
<!-- Standard images -->
<img src="user-avatar.png" alt="Sarah Johnson's profile picture" />
<img
  src="revenue-chart.svg"
  alt="Bar chart showing 25% revenue growth over 6 months"
/>

<!-- Decorative images -->
<img src="decorative-pattern.svg" alt="" role="presentation" />
<img src="divider-line.svg" alt="" />
```

#### Lazy Load Images

Implement lazy loading (e.g., `loading="lazy"` attribute or libraries like lazysizes) to defer offscreen image loading and improve performance.

**Examples**

HTML

```html
<img src="large-image.jpg" alt="Description" loading="lazy" />
```

ReactJS

```typescript
const ImageCard = ({ src, alt }: ImageCardProps): JSX.Element => {
    return (
        <img
            src={src}
            alt={alt}
            loading="lazy"
            decoding="async"
        />
    );
};
```

#### Use Modern Delivery Techniques

Consider using service workers or progressive image loading for advanced performance improvements.
