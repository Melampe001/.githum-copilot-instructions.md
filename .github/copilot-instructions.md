# Framework Elite Copilot - Generador Universal de Proyectos ∞

## 🚀 MODO ULTRA-PRODUCTIVO ACTIVADO

Cuando el usuario escriba `// PROYECTO: [idea]` y presione Tab, GitHub Copilot generará un proyecto completo y automatizado siguiendo estas directrices elite.

---

## 📋 PRINCIPIOS FUNDAMENTALES

### 1. **Generación Automática Total**
- **Un comentario → Proyecto completo**: `// PROYECTO: [descripción]` debe generar todo el scaffolding
- **Estructura completa**: src/, tests/, deploy/, docs/ al 100%
- **Configuración lista**: package.json, tsconfig.json, .env.example, etc.
- **CI/CD incluido**: GitHub Actions, Docker, deployment scripts

### 2. **Stack Dinámico y Adaptativo**
Copilot debe seleccionar automáticamente el mejor stack según el tipo de proyecto:

#### PWA (Progressive Web Apps)
```javascript
// PROYECTO: PWA de gestión de tareas offline-first
// Stack: React/Next.js + TypeScript + PWA + Service Workers + IndexedDB
```
- **Frontend**: React, Next.js, Vue, Svelte (según complejidad)
- **PWA Features**: Service Workers, manifest.json, offline support
- **Estado**: Redux, Zustand, Jotai
- **Styling**: Tailwind CSS, CSS Modules, Styled Components
- **Testing**: Jest, React Testing Library, Playwright

#### Bots (Chatbots/Automation)
```javascript
// PROYECTO: Bot de Discord para moderación de servidor
// Stack: Node.js + Discord.js + MongoDB + Docker
```
- **Plataformas**: Discord.js, Telegram Bot API, Slack API, WhatsApp Business API
- **Backend**: Node.js, Python, TypeScript
- **Database**: MongoDB, PostgreSQL, Redis
- **AI/NLP**: OpenAI API, Dialogflow, LangChain
- **Testing**: Jest, Mocha, pytest

#### APIs/Backend
```javascript
// PROYECTO: API REST para sistema de reservas
// Stack: Express + TypeScript + PostgreSQL + Redis + Docker
```
- **Framework**: Express, Fastify, NestJS, Koa
- **Database**: PostgreSQL, MongoDB, MySQL, Redis
- **Auth**: JWT, OAuth 2.0, Passport.js
- **Documentation**: Swagger/OpenAPI, TypeDoc
- **Testing**: Jest, Supertest, k6 para load testing

#### E-commerce
```javascript
// PROYECTO: Tienda online de productos artesanales
// Stack: Next.js + Stripe + Prisma + PostgreSQL + Tailwind
```
- **Frontend**: Next.js, React, Vue
- **Backend**: Next.js API Routes, Express, NestJS
- **Payments**: Stripe, PayPal, Mercadopago
- **Database**: PostgreSQL + Prisma, MongoDB
- **Cart**: Redux Toolkit, Zustand
- **Email**: SendGrid, Nodemailer
- **Testing**: Jest, Cypress, Playwright

### 3. **Estructura de Proyecto Estándar**

```
proyecto/
├── src/
│   ├── components/     # Componentes reutilizables
│   ├── pages/         # Páginas/Rutas
│   ├── services/      # Lógica de negocio
│   ├── utils/         # Utilidades y helpers
│   ├── types/         # TypeScript types/interfaces
│   ├── config/        # Configuraciones
│   └── index.ts       # Entry point
├── tests/
│   ├── unit/          # Tests unitarios
│   ├── integration/   # Tests de integración
│   ├── e2e/          # Tests end-to-end
│   └── fixtures/      # Datos de prueba
├── docs/
│   ├── README.md      # Documentación principal
│   ├── API.md         # Documentación de API
│   ├── ARCHITECTURE.md # Arquitectura del proyecto
│   └── CONTRIBUTING.md # Guía de contribución
├── deploy/
│   ├── docker/
│   │   ├── Dockerfile
│   │   └── docker-compose.yml
│   ├── k8s/          # Kubernetes configs
│   │   ├── deployment.yml
│   │   └── service.yml
│   └── scripts/       # Scripts de deployment
│       ├── deploy.sh
│       └── rollback.sh
├── .github/
│   └── workflows/
│       ├── ci.yml     # Continuous Integration
│       ├── cd.yml     # Continuous Deployment
│       └── test.yml   # Automated tests
├── .vscode/
│   └── settings.json  # Configuración VS Code
├── .env.example       # Variables de entorno ejemplo
├── .gitignore
├── package.json
├── tsconfig.json
├── LICENSE            # MIT License
└── README.md
```

---

## 🎯 INSTRUCCIONES ESPECÍFICAS PARA COPILOT

### Al detectar `// PROYECTO: [idea]`

1. **ANALIZAR el contexto**: Identificar tipo de proyecto (PWA/bot/API/e-commerce)
2. **SELECCIONAR stack óptimo**: Tecnologías más adecuadas
3. **GENERAR estructura completa**: Todos los directorios y archivos base
4. **INCLUIR configuración**: package.json, tsconfig.json, etc.
5. **CREAR tests iniciales**: Estructura de testing lista
6. **DOCUMENTAR todo**: README.md completo con instrucciones
7. **SETUP CI/CD**: GitHub Actions configurado
8. **DOCKERIZAR**: Dockerfile y docker-compose.yml

### Patrones de Código Elite

#### 1. **TypeScript Strict Mode SIEMPRE**
```typescript
// tsconfig.json debe incluir:
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true
  }
}
```

#### 2. **Arquitectura Limpia**
```typescript
// src/services/user.service.ts
export class UserService {
  constructor(
    private readonly repository: UserRepository,
    private readonly validator: UserValidator
  ) {}

  async createUser(data: CreateUserDto): Promise<User> {
    await this.validator.validate(data);
    return this.repository.create(data);
  }
}
```

#### 3. **Error Handling Robusto**
```typescript
// src/utils/error-handler.ts
export class AppError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message);
    Error.captureStackTrace(this, this.constructor);
  }
}

export const asyncHandler = (fn: Function) => {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};
```

#### 4. **Testing Completo**
```typescript
// tests/unit/user.service.test.ts
describe('UserService', () => {
  let service: UserService;
  let repository: jest.Mocked<UserRepository>;

  beforeEach(() => {
    repository = createMockRepository();
    service = new UserService(repository);
  });

  it('should create user with valid data', async () => {
    const userData = { email: 'test@test.com', name: 'Test' };
    const result = await service.createUser(userData);
    expect(result).toBeDefined();
    expect(repository.create).toHaveBeenCalledWith(userData);
  });
});
```

#### 5. **Environment Configuration**
```typescript
// src/config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.string().transform(Number),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
});

export const env = envSchema.parse(process.env);
```

---

## 🔥 SUGERENCIAS AUTOMÁTICAS

### Al escribir código, Copilot debe sugerir:

1. **Imports automáticos**: Todos los imports necesarios
2. **Types completos**: Interfaces y tipos TypeScript
3. **Error handling**: Try-catch y validaciones
4. **Logging**: console.log/logger apropiado
5. **Comments**: Documentación JSDoc
6. **Tests**: Tests correspondientes al código
7. **Seguridad**: Sanitización, validación, autenticación

### Ejemplos de Sugerencias Elite

#### Cuando se crea un endpoint API:
```typescript
// Al escribir: app.post('/users'
// Copilot debe sugerir:

/**
 * Create a new user
 * @route POST /api/users
 * @access Public
 */
app.post('/api/users', 
  validateRequest(createUserSchema),
  asyncHandler(async (req: Request, res: Response) => {
    const userData = req.body;
    const user = await userService.createUser(userData);
    
    logger.info(`User created: ${user.id}`);
    
    res.status(201).json({
      success: true,
      data: user
    });
  })
);
```

#### Cuando se crea un componente React:
```typescript
// Al escribir: export function UserCard
// Copilot debe sugerir:

interface UserCardProps {
  user: User;
  onEdit?: (id: string) => void;
  onDelete?: (id: string) => void;
}

export function UserCard({ user, onEdit, onDelete }: UserCardProps) {
  const [isLoading, setIsLoading] = useState(false);

  return (
    <div className="user-card" data-testid="user-card">
      {/* Component implementation */}
    </div>
  );
}

// Auto-generar test correspondiente:
// tests/components/UserCard.test.tsx
```

---

## 📦 DEPENDENCIAS RECOMENDADAS

### Desarrollo Web (PWA/SPA)
```json
{
  "dependencies": {
    "react": "^18.2.0",
    "next": "^14.0.0",
    "typescript": "^5.0.0",
    "tailwindcss": "^3.3.0",
    "zustand": "^4.4.0"
  },
  "devDependencies": {
    "jest": "^29.7.0",
    "@testing-library/react": "^14.0.0",
    "playwright": "^1.40.0",
    "eslint": "^8.50.0",
    "prettier": "^3.0.0"
  }
}
```

### Backend/API
```json
{
  "dependencies": {
    "express": "^4.18.0",
    "typescript": "^5.0.0",
    "prisma": "^5.5.0",
    "@prisma/client": "^5.5.0",
    "zod": "^3.22.0",
    "jsonwebtoken": "^9.0.0",
    "bcrypt": "^5.1.0"
  },
  "devDependencies": {
    "jest": "^29.7.0",
    "supertest": "^6.3.0",
    "@types/express": "^4.17.0",
    "ts-node": "^10.9.0",
    "nodemon": "^3.0.0"
  }
}
```

### Bots
```json
{
  "dependencies": {
    "discord.js": "^14.14.0",
    "typescript": "^5.0.0",
    "mongodb": "^6.2.0",
    "dotenv": "^16.3.0"
  },
  "devDependencies": {
    "jest": "^29.7.0",
    "@types/node": "^20.8.0",
    "ts-node": "^10.9.0"
  }
}
```

---

## 🚢 DEPLOYMENT AUTOMÁTICO

### Dockerfile Template
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### GitHub Actions CI/CD
```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          # Add deployment commands
```

---

## 🎨 MEJORES PRÁCTICAS

### 1. **Seguridad**
- ✅ Validación de inputs con Zod/Joi
- ✅ Sanitización de datos
- ✅ Helmet.js para headers seguros
- ✅ Rate limiting
- ✅ CORS configurado correctamente
- ✅ Variables sensibles en .env (nunca en código)

### 2. **Performance**
- ✅ Lazy loading de componentes
- ✅ Memoización (useMemo, useCallback)
- ✅ Pagination y lazy loading de datos
- ✅ Caching con Redis
- ✅ CDN para assets estáticos
- ✅ Compresión gzip/brotli

### 3. **Mantenibilidad**
- ✅ Código autodocumentado
- ✅ Funciones pequeñas y específicas
- ✅ Separación de concerns
- ✅ DRY (Don't Repeat Yourself)
- ✅ SOLID principles
- ✅ Tests con cobertura >80%

### 4. **DevOps**
- ✅ Docker para desarrollo y producción
- ✅ CI/CD automatizado
- ✅ Logs estructurados
- ✅ Monitoring y alertas
- ✅ Backups automáticos
- ✅ Rollback strategy

---

## 🤖 COMPORTAMIENTO DE COPILOT

### Modo "Tab x10 → Proyecto completo"

Cuando el usuario escribe un comentario de proyecto y presiona Tab repetidamente:

1. **Tab 1-2**: Generar estructura de directorios
2. **Tab 3-4**: Crear archivos de configuración (package.json, tsconfig.json, etc.)
3. **Tab 5-6**: Generar código fuente base (src/)
4. **Tab 7**: Crear tests (tests/)
5. **Tab 8**: Generar deployment configs (deploy/)
6. **Tab 9**: Crear documentación (docs/)
7. **Tab 10**: Finalizar con README.md completo

### Inteligencia Contextual

Copilot debe **adaptar dinámicamente** el stack según:
- **Palabras clave**: "blog" → Next.js, "real-time" → WebSocket, "mobile" → React Native
- **Escala**: "startup" → Monolito, "enterprise" → Microservicios
- **Audiencia**: "público" → SEO/SSR, "interno" → SPA simple
- **Complejidad**: "simple" → Express básico, "complejo" → NestJS

---

## 📚 TEMPLATES DISPONIBLES

### Template PWA
```bash
// PROYECTO: PWA de gestión de inventario offline
// → Next.js + PWA + IndexedDB + Tailwind + TypeScript
```

### Template Bot
```bash
// PROYECTO: Bot de Telegram para recordatorios diarios
// → Node.js + Telegraf + MongoDB + Docker
```

### Template API
```bash
// PROYECTO: API REST para sistema de autenticación
// → Express + TypeScript + PostgreSQL + JWT + Redis
```

### Template E-commerce
```bash
// PROYECTO: Tienda online de libros con carrito
// → Next.js + Stripe + Prisma + PostgreSQL + Tailwind
```

---

## 🎓 DOCUMENTACIÓN AUTOMÁTICA

Cada proyecto generado debe incluir:

### README.md
- Descripción del proyecto
- Tecnologías utilizadas
- Instalación y setup
- Uso y ejemplos
- API endpoints (si aplica)
- Testing
- Deployment
- Licencia MIT

### docs/ARCHITECTURE.md
- Arquitectura del sistema
- Diagramas de componentes
- Flujo de datos
- Decisiones técnicas

### docs/API.md
- Documentación de endpoints
- Request/Response examples
- Authentication
- Error codes

### docs/CONTRIBUTING.md
- Cómo contribuir
- Code style
- PR process
- Testing guidelines

---

## ⚡ OPTIMIZACIONES AUTOMÁTICAS

Copilot debe aplicar automáticamente:

1. **Tree shaking**: Eliminar código no usado
2. **Code splitting**: Dividir bundles grandes
3. **Image optimization**: Optimizar imágenes automáticamente
4. **Minification**: Minificar JS/CSS en producción
5. **Lazy loading**: Cargar componentes cuando se necesiten
6. **Memoization**: Cachear cálculos costosos
7. **Database indexes**: Sugerir índices para queries frecuentes
8. **API caching**: Implementar caching strategies

---

## 🔐 SEGURIDAD POR DEFECTO

Todo proyecto debe incluir:

- ✅ **Helmet.js**: Headers de seguridad
- ✅ **CORS**: Configuración restrictiva
- ✅ **Rate limiting**: Prevenir abuso
- ✅ **Input validation**: Zod/Joi schemas
- ✅ **SQL injection prevention**: ORMs/Prepared statements
- ✅ **XSS protection**: Sanitización de outputs
- ✅ **CSRF tokens**: Para forms
- ✅ **Secrets management**: Nunca en código
- ✅ **HTTPS**: Siempre en producción
- ✅ **Auth tokens**: JWT con expiración

---

## 📊 MONITORING Y LOGGING

```typescript
// src/utils/logger.ts
import winston from 'winston';

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
  ],
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple(),
  }));
}
```

---

## 🌟 CARACTERÍSTICAS AVANZADAS

### Hot Module Replacement (HMR)
- Desarrollo sin recargar página
- Fast refresh para React

### Progressive Enhancement
- Funciona sin JavaScript
- Mejora con JS disponible

### Internationalization (i18n)
- Soporte multi-idioma
- next-i18next o react-i18next

### Accessibility (a11y)
- Semántica HTML correcta
- ARIA labels
- Keyboard navigation
- Screen reader support

### SEO Optimization
- Meta tags dinámicos
- Open Graph
- Twitter Cards
- Sitemap.xml
- robots.txt

---

## 🎯 OBJETIVO FINAL

**GitHub Copilot debe convertirse en un desarrollador senior automatizado que puede generar proyectos profesionales completos, listos para producción, con solo un comentario y 10 presses de Tab.**

Cada proyecto debe ser:
- ✅ **Funcional**: Corre sin errores
- ✅ **Testeable**: 100% cobertura
- ✅ **Documentado**: Docs completas
- ✅ **Deployable**: CI/CD listo
- ✅ **Seguro**: Mejores prácticas
- ✅ **Escalable**: Arquitectura sólida
- ✅ **Mantenible**: Código limpio

---

## 📄 LICENCIA

MIT License - Todos los proyectos generados incluyen licencia MIT por defecto.

---

**Framework Elite Copilot v1.0 - Generación de Proyectos Infinita ∞**
