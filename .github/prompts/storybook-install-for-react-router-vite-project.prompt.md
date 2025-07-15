---
mode: 'agent'
model: Claude Sonnet 4
description: 'a prompt for installing and configuring the latest storybook in a react-router vite plugin project.'
---

# Storybook Installation Guide for React Router v7 + Vite Projects

This guide provides step-by-step instructions for installing and configuring Storybook in a React Router v7 project that uses the Vite plugin.

## Installation Steps

### 1. Install Storybook

Run the Storybook installation command and select the recommended configuration:

```bash
npm create storybook@latest
```

When prompted, select: **"Recommended: Component dev, docs, test"**

### 2. Update Main Vite Configuration

Update your main `vite.config.ts` file to conditionally load plugins based on environment to prevent React Router plugin conflicts:

```typescript
// Add this condition near the top of the file
const isStorybook = process.env.NODE_ENV === 'storybook' || process.argv[1]?.includes('storybook') || process.env.VITEST;

// Update the plugins array to use conditional loading
export default defineConfig({
  plugins: isStorybook 
    ? [tsconfigPaths()] 
    : [tailwindcss(), reactRouter(), tsconfigPaths()],
  // ...rest of your config
});
```

### 3. Create Storybook Vite Configuration

Create or update `.storybook/vite.config.ts` with a minimal configuration:

```typescript
import { defineConfig } from 'vite';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [tsconfigPaths()],
  resolve: {
    alias: {
      '@': '/app',
    },
  },
});
```

### 4. Update Storybook Main Configuration

Update `.storybook/main.ts` to include your app components in the stories array:

```typescript
const config: StorybookConfig = {
  "stories": [
    "../stories/**/*.mdx",
    "../stories/**/*.stories.@(js|jsx|mjs|ts|tsx)",
    "../app/components/**/*.stories.@(js|jsx|mjs|ts|tsx)" // Add this line
  ],
  // ...rest of your config
};
```

### 5. Import App Styles

Update `.storybook/preview.ts` to import your app's CSS for Tailwind styling:

```typescript
import type { Preview } from '@storybook/react-vite'
import '../app/app.css' // Add this import

const preview: Preview = {
  // ...rest of your config
};
```

### 6. Install Required Utilities

Install the utility packages if they're not already present:

```bash
npm install clsx tailwind-merge
```

### 7. Create Utility Function

Create a `cn` utility function in `app/utils/cn.ts` for combining CSS classes:

```typescript
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### 8. Test the Setup

Start Storybook to verify everything is working:

```bash
npm run storybook
```

Storybook should start at [http://localhost:6006](http://localhost:6006)

### 9. Verify Vitest Integration

Test the Vitest integration for story testing:

```bash
npx vitest --project=storybook --run
```

## Key Points

- **Main Issue**: The React Router Vite plugin conflicts with Storybook, so conditional plugin loading is essential
- **Minimal Config**: The `.storybook/vite.config.ts` should be kept minimal with just TypeScript paths
- **Styling**: Import `app.css` in `preview.ts` to ensure Tailwind styles work in Storybook
- **Features**: This setup enables component development, documentation, testing, and accessibility checking while maintaining compatibility with React Router v7

## Available Commands

After installation, you'll have these new commands available:

```bash
npm run storybook          # Start Storybook development server
npm run build-storybook    # Build Storybook for production
npx vitest --project=storybook --run  # Run story tests
```

## Troubleshooting

If you encounter issues:

1. Ensure the conditional plugin loading is correctly implemented in `vite.config.ts`
2. Verify that `.storybook/vite.config.ts` exists and has the minimal configuration
3. Check that `app.css` is properly imported in `.storybook/preview.ts`
4. Make sure all required dependencies are installed

This setup provides a robust foundation for component development and testing within your React Router v7 project.
