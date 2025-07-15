---
mode: 'agent'
model: Claude Sonnet 4
description: 'a prompt for converting project file structure following bulletproof-react method.'
---

# Bulletproof-React Project Restructuring Guide

This document serves as a comprehensive prompt and guide for converting existing React projects to follow the bulletproof-react methodology. Use this as a complete reference when working with an LLM to restructure your React applications.

## 📋 Project Analysis Checklist

Before starting the restructuring process, analyze the current project:

### 1. **Current Structure Assessment**
- [ ] List all components and their current locations
- [ ] Identify feature domains (e.g., users, products, auth, etc.)
- [ ] Check for existing folder organization patterns
- [ ] Identify shared vs feature-specific components
- [ ] Review existing API structure
- [ ] Check for barrel exports (index.js files)
- [ ] Assess TypeScript usage and type organization

### 2. **Project Domain Identification**
Ask these questions to identify features:
- What are the main business domains in this application?
- Which components belong together functionally?
- What are the main user journeys/workflows?
- Which components are truly shared vs feature-specific?

## 🏗️ Bulletproof-React Structure Template

Use this as the target structure for any React project:

```
src/ (or app/ for Next.js/React Router)
├── api/                    # Global API client and utilities
│   ├── client.ts          # Base HTTP client configuration
│   └── types.ts           # Global API types
├── components/            # Shared components used across features
│   ├── ui/               # Generic UI components (Button, Modal, etc.)
│   ├── layout/           # Layout components (Header, Sidebar, etc.)
│   └── forms/            # Shared form components (optional)
├── config/               # Global configurations
│   └── index.ts          # Environment variables, app settings
├── features/             # Feature-based modules (CORE PRINCIPLE)
│   ├── feature-name/     # Each business domain gets its own folder
│   │   ├── api/         # Feature-specific API calls
│   │   ├── components/  # Feature-specific components
│   │   ├── hooks/       # Feature-specific hooks (optional)
│   │   ├── types/       # Feature-specific TypeScript types
│   │   ├── utils/       # Feature-specific utilities (optional)
│   │   └── index.ts     # Feature exports (avoid barrel exports)
├── hooks/               # Shared hooks used across features
│   └── index.ts         # useAsync, useLocalStorage, etc.
├── lib/                 # Reusable libraries and utilities
│   ├── utils.ts         # General utilities
│   └── cn.ts           # className utility (for Tailwind)
├── stores/              # Global state management (optional)
├── types/               # Global TypeScript types
│   └── index.ts         # Shared interfaces and types
└── utils/               # Shared utility functions
```

## 🎯 Core Principles to Follow

### 1. **Feature-Based Organization**
- Each major functionality becomes a self-contained feature folder
- Features should represent business domains, not technical layers
- Examples: `auth`, `users`, `products`, `dashboard`, `settings`

### 2. **Unidirectional Dependencies**
```
Shared (components, hooks, lib, types, utils)
  ↓ (can import from)
Features (feature-specific code)
  ↓ (can import from)
App (routes, main application)
```

### 3. **No Cross-Feature Imports**
- Features cannot import from other features
- Compose features at the application/route level
- Use shared components/hooks for common functionality

### 4. **Component Hierarchy Rules**
- **`/components/ui`**: Generic, reusable UI primitives
- **`/components/layout`**: Application layout components
- **`/features/*/components`**: Feature-specific components only
- **Never** put feature-specific components in shared folders

## 🔄 Step-by-Step Restructuring Process

### Phase 1: Analysis and Planning
1. **Map Current Components to Features**
   ```
   Current: components/UserProfile.tsx
   Target:  features/users/components/UserProfile.tsx
   
   Current: components/ProductCard.tsx  
   Target:  features/products/components/ProductCard.tsx
   
   Current: components/Button.tsx
   Target:  components/ui/Button.tsx (shared)
   ```

2. **Identify Shared vs Feature Components**
   - **Shared**: Button, Modal, Input, Card, Layout components
   - **Feature-specific**: UserProfile, ProductList, OrderSummary

### Phase 2: Create New Structure
```bash
# Create main folders
mkdir -p src/{api,components/{ui,layout},config,features,hooks,lib,types,utils}

# Create feature folders (example)
mkdir -p src/features/{users,products,orders}/{api,components,types}
```

### Phase 3: Move Components
**Critical Rules:**
- Move feature-specific components to their respective feature folders
- Keep generic UI components in `components/ui/`
- Update all import paths after moving files
- Test after each major move to ensure nothing breaks

### Phase 4: Create API Structure
```typescript
// src/api/client.ts - Base client
export class ApiClient {
  // HTTP client with interceptors, error handling
}

// src/features/users/api/users.ts - Feature API
import { apiClient } from '~/api/client';
export const usersApi = {
  getAll: () => apiClient.get('/users'),
  // ...
};
```

### Phase 5: Type Organization
```typescript
// src/types/index.ts - Global types
export interface ApiResponse<T> {
  data: T;
  success: boolean;
  error?: string;
}

// src/features/users/types/index.ts - Feature types
export interface User {
  id: string;
  name: string;
  // ...
}
```

## 🚨 Common Mistakes to Avoid

### 1. **Wrong Component Placement**
```typescript
// ❌ DON'T: Put feature components in shared folders
src/components/UserDashboard.tsx

// ✅ DO: Put feature components in feature folders
src/features/users/components/UserDashboard.tsx
```

### 2. **Cross-Feature Imports**
```typescript
// ❌ DON'T: Import between features
import { UserCard } from '~/features/users/components/UserCard';
// Used inside features/products/

// ✅ DO: Compose at app level
// In routes or main app components
import { UserCard } from '~/features/users';
import { ProductList } from '~/features/products';
```

### 3. **Barrel Export Overuse**
```typescript
// ❌ AVOID: Complex barrel exports (bad for tree shaking)
export * from './component1';
export * from './component2';

// ✅ PREFER: Direct imports
import { SpecificComponent } from '~/features/users/components/SpecificComponent';
```

### 4. **Wrong Feature Granularity**
```
❌ Too granular: features/user-profile/, features/user-settings/
✅ Right size:   features/users/ (with profile and settings components)

❌ Too broad:    features/admin/ (everything admin-related)
✅ Better:       features/user-management/, features/system-settings/
```

## 🛠️ LLM Restructuring Instructions

When working with an LLM to restructure a project, use this exact prompt structure:

### Initial Analysis Prompt
```
I want to restructure my React project to follow bulletproof-react methodology. 

Current project structure:
[PASTE CURRENT FOLDER STRUCTURE]

The project is about: [DESCRIBE PROJECT DOMAIN]

Main components include: [LIST MAIN COMPONENTS]

Please:
1. Analyze the current structure
2. Identify the main feature domains
3. Create a bulletproof-react folder structure plan
4. Show me which components should move where
5. Identify shared vs feature-specific components
```

### Implementation Prompt
```
Now implement the bulletproof-react restructuring:

1. Create the new folder structure following bulletproof-react principles
2. Move components to appropriate locations:
   - Feature-specific components → features/[domain]/components/
   - Generic UI components → components/ui/
   - Layout components → components/layout/
3. Create API structure with base client and feature APIs
4. Set up proper TypeScript types organization
5. Create shared hooks in /hooks
6. Move utilities to /lib
7. Update import paths as needed
8. Create index files for features (but prefer direct imports)

Follow these rules:
- No cross-feature imports
- Unidirectional dependencies
- Feature independence
- Proper component hierarchy
```

## 📝 Documentation Updates

After restructuring, always update:

### 1. **README.md**
Include sections for:
- Project structure explanation
- Architecture principles
- Development guidelines
- Component placement rules
- Contributing guidelines

### 2. **PROJECT_STRUCTURE.md**
Detailed architecture documentation with:
- Folder explanations
- Import/export guidelines
- Feature development process
- Examples of correct patterns

### 3. **ESLint Rules**
```javascript
// Add these rules to prevent architecture violations
'import/no-restricted-paths': [
  'error',
  {
    zones: [
      // Prevent cross-feature imports
      {
        target: './src/features/users',
        from: './src/features',
        except: ['./users'],
      },
      // Enforce unidirectional flow
      {
        target: [
          './src/components',
          './src/hooks',
          './src/lib',
          './src/types',
          './src/utils',
        ],
        from: ['./src/features', './src/app'],
      },
    ],
  },
],
```

## 🔍 Quality Checks

After restructuring, verify:

- [ ] No cross-feature imports exist
- [ ] All imports are updated and working
- [ ] Features are self-contained
- [ ] Shared components are truly generic
- [ ] API layer is properly organized
- [ ] Types are in correct locations
- [ ] Application still builds and runs
- [ ] No circular dependencies
- [ ] ESLint rules are enforced

## 📚 Framework-Specific Notes

### React Router v7
- Use `app/` instead of `src/`
- Leverage file-based routing in `app/routes/`
- Use React Router's data loading patterns

### Next.js
- Use `src/` or root-level organization
- Respect Next.js conventions for pages/app directory
- Adapt API routes structure accordingly

### Vite/CRA
- Standard `src/` structure
- Ensure proper alias configuration for clean imports

## 🎯 Success Metrics

A successful bulletproof-react restructuring achieves:

1. **Clear Feature Boundaries**: Each feature is self-contained
2. **Scalable Architecture**: Easy to add new features without affecting existing ones
3. **Developer Experience**: New developers can easily understand the structure
4. **Maintainability**: Changes in one feature don't break others
5. **Testability**: Features can be tested in isolation
6. **Type Safety**: Comprehensive TypeScript coverage

## 🔗 Additional Resources

- [Bulletproof React Repository](https://github.com/alan2207/bulletproof-react)
- [Project Structure Documentation](https://github.com/alan2207/bulletproof-react/blob/master/docs/project-structure.md)
- [Architecture Principles](https://github.com/alan2207/bulletproof-react/blob/master/docs/project-standards.md)

---

Use this guide as a complete reference when working with LLMs to restructure React projects. The key is systematic analysis, careful planning, and adherence to bulletproof-react principles throughout the process.
