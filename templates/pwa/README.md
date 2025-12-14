# PWA Template - Progressive Web App

## 🚀 Ejemplo de uso

```typescript
// PROYECTO: PWA de gestión de tareas offline-first
// Stack: Next.js + TypeScript + PWA + Service Workers + IndexedDB
```

## 📦 Stack Generado

- **Frontend**: Next.js 14 con App Router
- **Lenguaje**: TypeScript 5.x
- **PWA**: next-pwa plugin
- **Estado**: Zustand / Redux Toolkit
- **Styling**: Tailwind CSS
- **Storage**: IndexedDB para offline
- **Testing**: Jest + React Testing Library + Playwright
- **Linting**: ESLint + Prettier

## 📁 Estructura

```
pwa-project/
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── api/
│   ├── components/
│   │   ├── ui/
│   │   └── features/
│   ├── lib/
│   │   ├── db.ts          # IndexedDB wrapper
│   │   └── sync.ts        # Sync logic
│   ├── hooks/
│   ├── store/
│   └── types/
├── public/
│   ├── manifest.json
│   ├── icons/
│   └── sw.js
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── next.config.js
├── tsconfig.json
└── package.json
```

## 🎯 Características Incluidas

### Service Worker
- Caching estratégico
- Offline fallback
- Background sync
- Push notifications

### IndexedDB
- CRUD operations
- Sync queue
- Conflict resolution

### Manifest.json
```json
{
  "name": "PWA App",
  "short_name": "PWA",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

### Next.js Config
```javascript
const withPWA = require('next-pwa')({
  dest: 'public',
  register: true,
  skipWaiting: true,
  disable: process.env.NODE_ENV === 'development'
});

module.exports = withPWA({
  reactStrictMode: true,
  swcMinify: true,
});
```

## 📦 Dependencies

```json
{
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^5.0.0",
    "zustand": "^4.4.0",
    "idb": "^7.1.1",
    "next-pwa": "^5.6.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/node": "^20.0.0",
    "tailwindcss": "^3.3.0",
    "postcss": "^8.4.0",
    "autoprefixer": "^10.4.0",
    "jest": "^29.7.0",
    "@testing-library/react": "^14.0.0",
    "playwright": "^1.40.0",
    "eslint": "^8.50.0",
    "prettier": "^3.0.0"
  }
}
```

## 🚀 Scripts

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "jest",
    "test:e2e": "playwright test",
    "type-check": "tsc --noEmit"
  }
}
```

## 🧪 Tests Incluidos

### Unit Test Example
```typescript
// tests/unit/components/TaskList.test.tsx
import { render, screen } from '@testing-library/react';
import TaskList from '@/components/TaskList';

describe('TaskList', () => {
  it('renders tasks correctly', () => {
    const tasks = [{ id: '1', title: 'Test Task' }];
    render(<TaskList tasks={tasks} />);
    expect(screen.getByText('Test Task')).toBeInTheDocument();
  });
});
```

### E2E Test Example
```typescript
// tests/e2e/pwa.spec.ts
import { test, expect } from '@playwright/test';

test('PWA can be installed', async ({ page }) => {
  await page.goto('/');
  await page.waitForEvent('console', msg => 
    msg.text().includes('service worker registered')
  );
});
```

## 🐳 Docker

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/public ./public
EXPOSE 3000
CMD ["npm", "start"]
```

## 🌐 Deploy

### Vercel (Recomendado)
```bash
npm install -g vercel
vercel --prod
```

### Docker
```bash
docker build -t pwa-app .
docker run -p 3000:3000 pwa-app
```

## 📱 Testing PWA Features

1. **Install Prompt**: Navigate to site on mobile
2. **Offline Mode**: Disable network in DevTools
3. **Add to Home Screen**: Chrome menu → Install
4. **Push Notifications**: Grant permissions

## ✅ Checklist de Funcionalidades

- [x] Service Worker registrado
- [x] Manifest.json válido
- [x] Iconos en múltiples tamaños
- [x] Offline fallback
- [x] IndexedDB configurado
- [x] Background sync
- [x] Responsive design
- [x] Tests E2E
- [x] Lighthouse score >90
