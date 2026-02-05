# Frontend Projects

This section covers frontend web applications and user interface projects.

## Project Overview

| Project | Description | Stack | Platform |
|---------|-------------|-------|----------|
| [Shotgun Socials Poster](shotgun-socials-poster.md) | Multi-platform social media poster | React/TypeScript/Vite | Web |

## Categories

### Social Media Tools

Tools for managing social media content and engagement.

<div class="grid cards" markdown>

- :material-bullhorn:{ .lg .middle } **Shotgun Socials Poster**

    ---

    A React-based web app that lets users compose a single post and publish
    it to multiple social media platforms at once.

    [:octicons-arrow-right-24: Learn more](shotgun-socials-poster.md)

</div>

## Common Patterns

### React Best Practices

All React projects follow consistent patterns:

```typescript
// Component structure
import { useState, useEffect } from 'react';
import type { FC } from 'react';

interface Props {
  title: string;
  onSubmit: (data: FormData) => void;
}

export const Component: FC<Props> = ({ title, onSubmit }) => {
  const [state, setState] = useState<string>('');
  
  useEffect(() => {
    // Side effects
  }, []);
  
  return <div>{/* JSX */}</div>;
};
```

### TypeScript Configuration

Consistent TypeScript setup:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

### Vite Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    outDir: 'dist',
    sourcemap: true
  }
});
```

## Quick Links

- [Shotgun Socials Poster :material-arrow-right:](shotgun-socials-poster.md)
