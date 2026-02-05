# Shotgun Socials Poster

[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-3178C6?logo=typescript)](https://www.typescriptlang.org/)
[![React](https://img.shields.io/badge/React-18-61DAFB?logo=react)](https://reactjs.org/)
[![Vite](https://img.shields.io/badge/Vite-5-646CFF?logo=vite)](https://vitejs.dev/)
[![GitHub Pages](https://img.shields.io/badge/Deployed%20on-GitHub%20Pages-222?logo=github)](https://pages.github.com/)

A React-based web app that lets users compose a single post and publish it to multiple
social media platforms at once — supporting Facebook, Instagram, Twitter/X, Threads,
BlueSky, Reddit, TikTok, and Discord with platform-specific validation and preview.

[:fontawesome-brands-github: View on GitHub](https://github.com/riyanimam/shotgun-socials-poster){ .md-button }
[:material-web: Live Demo](https://riyanimam.github.io/shotgun-socials-poster){ .md-button }

## Features

- **Multi-Platform Publishing**: Post to 8+ social platforms simultaneously
- **Platform-Specific Validation**: Character limits, hashtag rules, mentions
- **Real-Time Preview**: See how your post looks on each platform
- **Rich Text Editor**: Format text with bold, italic, links
- **Image Support**: Upload and preview images for posts
- **Draft Saving**: Save drafts locally in browser
- **Responsive Design**: Works on desktop, tablet, and mobile
- **GitHub Pages Deployment**: Free hosting included

## Supported Platforms

| Platform | Character Limit | Special Features |
|----------|----------------|------------------|
| **Twitter/X** | 280 | Threads, hashtags, mentions |
| **Facebook** | 63,206 | Rich previews, tagging |
| **Instagram** | 2,200 | Hashtags (30 max), mentions |
| **Threads** | 500 | Text-focused, links |
| **BlueSky** | 300 | Decentralized, links |
| **Reddit** | 40,000 | Subreddit selection |
| **TikTok** | 2,200 | Video description |
| **Discord** | 2,000 | Embeds, webhooks |

## Prerequisites

- **Node.js** >= 20
- **npm** or **pnpm**
- Modern web browser

## Quick Start

```bash
# Clone the repository
git clone https://github.com/riyanimam/shotgun-socials-poster.git
cd shotgun-socials-poster

# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

## Project Structure

```text
shotgun-socials-poster/
├── src/
│   ├── components/
│   │   ├── Editor/
│   │   │   ├── TextEditor.tsx       # Rich text editor
│   │   │   └── ImageUploader.tsx    # Image upload
│   │   ├── PlatformSelector/
│   │   │   └── PlatformSelector.tsx # Platform checkboxes
│   │   ├── Preview/
│   │   │   ├── PreviewCard.tsx      # Platform preview
│   │   │   └── PlatformPreviews.tsx # All previews
│   │   └── Publish/
│   │       └── PublishButton.tsx    # Publish action
│   ├── hooks/
│   │   ├── useDraft.ts              # Draft management
│   │   ├── useValidation.ts         # Post validation
│   │   └── usePublish.ts            # Publishing logic
│   ├── types/
│   │   └── platforms.ts             # Type definitions
│   ├── utils/
│   │   ├── validation.ts            # Validation rules
│   │   └── formatting.ts            # Text formatting
│   ├── App.tsx                      # Main app component
│   └── main.tsx                     # Entry point
├── public/
│   └── platforms/                   # Platform icons
├── index.html
├── vite.config.ts
├── tsconfig.json
└── package.json
```

## Usage

### Composing a Post

1. **Write your content** in the text editor
2. **Select platforms** you want to post to
3. **Preview** how it looks on each platform
4. **Adjust** content for platform-specific requirements
5. **Publish** to all selected platforms

### Platform-Specific Tips

#### Twitter/X

```
Keep it under 280 characters!
Use #hashtags for discovery
Tag with @mentions
Break long posts into threads
```

#### Instagram

```
First 125 chars appear in feed
Use 5-10 relevant #hashtags
Tag users with @username
Add location for visibility
```

#### Reddit

```
Choose appropriate subreddit
Follow subreddit rules
Use descriptive titles
Engage in comments
```

#### Discord

```
Use webhooks for automation
Format with markdown
Include rich embeds
Mention roles: @everyone
```

## Configuration

### Platform Validation Rules

```typescript
// src/utils/validation.ts
export const platformRules = {
  twitter: {
    maxLength: 280,
    maxHashtags: 10,
    allowsImages: true,
    allowsVideos: true
  },
  instagram: {
    maxLength: 2200,
    maxHashtags: 30,
    allowsImages: true,
    allowsVideos: true
  },
  facebook: {
    maxLength: 63206,
    maxHashtags: Infinity,
    allowsImages: true,
    allowsVideos: true
  },
  // ... more platforms
};
```

### Custom Hooks

#### useDraft Hook

```typescript
// src/hooks/useDraft.ts
import { useState, useEffect } from 'react';

export function useDraft() {
  const [draft, setDraft] = useState<string>('');

  // Auto-save to localStorage
  useEffect(() => {
    const timer = setTimeout(() => {
      localStorage.setItem('draft', draft);
    }, 1000);

    return () => clearTimeout(timer);
  }, [draft]);

  // Load draft on mount
  useEffect(() => {
    const saved = localStorage.getItem('draft');
    if (saved) setDraft(saved);
  }, []);

  return [draft, setDraft] as const;
}
```

#### useValidation Hook

```typescript
// src/hooks/useValidation.ts
import { useMemo } from 'react';
import { platformRules } from '../utils/validation';

export function useValidation(text: string, platform: string) {
  return useMemo(() => {
    const rules = platformRules[platform];

    return {
      isValid: text.length <= rules.maxLength,
      charCount: text.length,
      remaining: rules.maxLength - text.length,
      hashtags: (text.match(/#\w+/g) || []).length
    };
  }, [text, platform]);
}
```

## Components

### TextEditor Component

```typescript
// src/components/Editor/TextEditor.tsx
import { FC } from 'react';

interface TextEditorProps {
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
}

export const TextEditor: FC<TextEditorProps> = ({
  value,
  onChange,
  placeholder = 'What\'s on your mind?'
}) => {
  return (
    <div className="editor">
      <textarea
        value={value}
        onChange={(e) => onChange(e.target.value)}
        placeholder={placeholder}
        className="editor-textarea"
        rows={6}
      />
      <div className="editor-toolbar">
        <button>Bold</button>
        <button>Italic</button>
        <button>Link</button>
      </div>
    </div>
  );
};
```

### PlatformPreview Component

```typescript
// src/components/Preview/PreviewCard.tsx
import { FC } from 'react';

interface PreviewProps {
  platform: string;
  content: string;
  image?: string;
}

export const PreviewCard: FC<PreviewProps> = ({
  platform,
  content,
  image
}) => {
  return (
    <div className={`preview-card ${platform}`}>
      <div className="preview-header">
        <img src={`/platforms/${platform}.svg`} alt={platform} />
        <span>{platform}</span>
      </div>
      <div className="preview-content">
        {image && <img src={image} alt="Post" />}
        <p>{content}</p>
      </div>
    </div>
  );
};
```

## Styling

### Tailwind CSS

```html
<!-- Example styling with Tailwind -->
<div class="max-w-4xl mx-auto p-6">
  <div class="bg-white rounded-lg shadow-md p-6">
    <h2 class="text-2xl font-bold mb-4">Compose Post</h2>
    <textarea
      class="w-full border rounded-lg p-4 focus:ring-2 focus:ring-blue-500"
      rows="6"
    ></textarea>
  </div>
</div>
```

### CSS Modules

```css
/* Editor.module.css */
.editor {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
}

.editorTextarea {
  width: 100%;
  border: none;
  outline: none;
  font-size: 16px;
  resize: vertical;
}

.editorToolbar {
  display: flex;
  gap: 8px;
  margin-top: 12px;
  border-top: 1px solid #eee;
  padding-top: 12px;
}
```

## Deployment

### GitHub Pages

```bash
# Build for production
npm run build

# Deploy to GitHub Pages (automated in CI/CD)
npm run deploy
```

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

## Gotchas & Tips

!!! warning "Character Counting"
    Different platforms count characters differently. URLs may be shortened
    on Twitter, emojis may count as multiple characters.
    ```typescript
    // Twitter-specific counting
    const twitterCount = text.replace(/https?:\/\/\S+/g, 'x'.repeat(23)).length;
    ```

!!! tip "Image Optimization"
    Compress images before uploading to meet platform requirements:
    ```typescript
    async function compressImage(file: File): Promise<File> {
      // Use browser-image-compression or similar
      const options = { maxSizeMB: 1, maxWidthOrHeight: 1920 };
      return await imageCompression(file, options);
    }
    ```

!!! note "API Integration"
    This is a frontend-only demo. For actual posting, you'd need:
    - Backend server for OAuth flows
    - Platform API keys and secrets
    - Token management
    - Rate limiting

!!! warning "CORS Issues"
    Direct API calls from browser will hit CORS. Use a backend proxy:
    ```typescript
    // Instead of direct API call
    fetch('<https://api.twitter.com/>...') // ❌ CORS error

    // Use your backend
    fetch('https://your-backend.com/api/post') // ✓
    ```

!!! tip "Local Storage Limits"
    Browser localStorage has 5-10MB limit. For larger drafts, consider:
    - IndexedDB for larger storage
    - Cloud sync with Firebase/Supabase
    - Compression before storage

## Troubleshooting

### Build Failures

```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install

# Clear Vite cache
rm -rf node_modules/.vite
npm run dev
```

### Preview Not Updating

```typescript
// Ensure proper state updates
const [content, setContent] = useState('');

// Use functional updates for complex state
setContent(prev => prev + newText);

// Force re-render if needed
const [key, setKey] = useState(0);
<Preview key={key} content={content} />
```

### GitHub Pages 404

```javascript
// vite.config.ts - set correct base path
export default defineConfig({
  base: '/shotgun-socials-poster/', // Your repo name
  plugins: [react()]
});
```

## Enhancement Ideas

- [ ] Add OAuth integration for actual posting
- [ ] Implement post scheduling
- [ ] Add analytics dashboard
- [ ] Support for video uploads
- [ ] Add URL shortener integration
- [ ] Implement hashtag suggestions
- [ ] Add emoji picker
- [ ] Create browser extension
- [ ] Add team collaboration features
- [ ] Implement A/B testing for posts

## Resources

- [React Documentation](https://react.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Vite Guide](https://vitejs.dev/guide/)
- [Twitter API](https://developer.twitter.com/en/docs)
- [Facebook Graph API](https://developers.facebook.com/docs/graph-api/)
- [Instagram API](https://developers.facebook.com/docs/instagram-api/)
- [Reddit API](https://www.reddit.com/dev/api/)
- [Discord Webhooks](https://discord.com/developers/docs/resources/webhook)
