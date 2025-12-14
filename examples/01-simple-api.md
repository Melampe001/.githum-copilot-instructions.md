# Ejemplo 1: API REST Básica

## 📝 Descripción

Una API REST simple para gestión de tareas (TODO list) con autenticación JWT y base de datos PostgreSQL.

**Nivel**: ⭐ Principiante  
**Tiempo estimado**: 5-10 minutos  
**Líneas de código generadas**: ~500

## 🚀 Comentario de Proyecto

Copia y pega este comentario en un nuevo archivo `.ts` o `.js`:

```typescript
// PROYECTO: API REST básica para gestión de tareas con autenticación JWT
```

O con más detalles:

```typescript
// PROYECTO: API REST de TODO list
// FEATURES: CRUD de tareas, autenticación JWT, validación de datos
// STACK: Express + TypeScript + PostgreSQL + Prisma
```

## 📦 Stack Generado

- **Backend**: Express.js 4.x
- **Lenguaje**: TypeScript 5.x
- **Database**: PostgreSQL 16
- **ORM**: Prisma 5.x
- **Auth**: JWT + bcrypt
- **Validation**: Zod
- **Testing**: Jest + Supertest
- **Docker**: Incluido

## 📁 Estructura Esperada

```
api-todo/
├── src/
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   └── task.controller.ts
│   ├── services/
│   │   ├── auth.service.ts
│   │   └── task.service.ts
│   ├── middleware/
│   │   ├── auth.middleware.ts
│   │   └── validation.middleware.ts
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   └── task.routes.ts
│   ├── types/
│   │   └── index.ts
│   ├── utils/
│   │   ├── logger.ts
│   │   └── errors.ts
│   └── index.ts
├── prisma/
│   └── schema.prisma
├── tests/
│   ├── auth.test.ts
│   └── tasks.test.ts
├── Dockerfile
├── docker-compose.yml
├── package.json
├── tsconfig.json
└── .env.example
```

## 🎯 Endpoints Generados

### Autenticación

- `POST /api/auth/register` - Registrar usuario
- `POST /api/auth/login` - Login
- `POST /api/auth/logout` - Logout

### Tareas

- `GET /api/tasks` - Listar tareas (con paginación)
- `GET /api/tasks/:id` - Obtener tarea por ID
- `POST /api/tasks` - Crear tarea
- `PUT /api/tasks/:id` - Actualizar tarea
- `DELETE /api/tasks/:id` - Eliminar tarea

## ⚙️ Características Incluidas

✅ Autenticación JWT  
✅ Hash de passwords (bcrypt)  
✅ Validación de inputs (Zod)  
✅ Manejo de errores centralizado  
✅ Logging con Winston  
✅ Rate limiting  
✅ CORS configurado  
✅ Helmet security headers  
✅ Prisma migrations  
✅ Tests unitarios e integración  
✅ Docker y docker-compose  
✅ Variables de entorno

## 🔧 Pasos Siguientes

### 1. Instalar Dependencias

```bash
npm install
```

### 2. Configurar Base de Datos

```bash
# Copiar variables de entorno
cp .env.example .env

# Editar .env con tus credenciales
DATABASE_URL="postgresql://user:password@localhost:5432/todos"
JWT_SECRET="tu-secreto-aqui"
```

### 3. Ejecutar Migraciones

```bash
npx prisma migrate dev
```

### 4. Ejecutar en Desarrollo

```bash
npm run dev
```

La API estará disponible en `http://localhost:3000`

### 5. Ejecutar Tests

```bash
npm test
```

### 6. Usar Docker (Alternativa)

```bash
docker-compose up -d
```

## 📊 Ejemplo de Uso

### Registrar Usuario

```bash
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123",
    "name": "John Doe"
  }'
```

### Login

```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123"
  }'
```

Respuesta:
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "name": "John Doe"
    }
  }
}
```

### Crear Tarea

```bash
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "title": "Completar proyecto",
    "description": "Terminar la API REST",
    "status": "pending"
  }'
```

### Listar Tareas

```bash
curl http://localhost:3000/api/tasks \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## 🎨 Personalización

### Agregar Categorías

Modifica `prisma/schema.prisma`:

```prisma
model Category {
  id    String @id @default(uuid())
  name  String
  tasks Task[]
}

model Task {
  // ... campos existentes
  categoryId String?
  category   Category? @relation(fields: [categoryId], references: [id])
}
```

### Agregar Prioridades

Agrega un enum en el schema:

```prisma
enum Priority {
  LOW
  MEDIUM
  HIGH
  URGENT
}

model Task {
  // ... campos existentes
  priority Priority @default(MEDIUM)
}
```

### Agregar Fechas de Vencimiento

```prisma
model Task {
  // ... campos existentes
  dueDate DateTime?
}
```

## 🐛 Troubleshooting

### Error: Cannot connect to database

```bash
# Verifica que PostgreSQL esté corriendo
docker ps

# O inicia los servicios
docker-compose up -d postgres
```

### Error: Prisma Client not generated

```bash
npx prisma generate
```

### Tests fallan

```bash
# Usa una base de datos de test
DATABASE_URL="postgresql://user:password@localhost:5432/todos_test" npm test
```

## 🚀 Deploy

### Heroku

```bash
heroku create
heroku addons:create heroku-postgresql:hobby-dev
git push heroku main
```

### Railway

```bash
railway init
railway up
```

### Docker

```bash
docker build -t todo-api .
docker run -p 3000:3000 --env-file .env todo-api
```

## 📈 Mejoras Sugeridas

1. **Agregar paginación avanzada** con cursor-based pagination
2. **Implementar búsqueda** full-text con PostgreSQL
3. **Agregar filtros** por status, categoría, fecha
4. **Implementar notificaciones** por email
5. **Agregar tareas recurrentes**
6. **Implementar compartir tareas** entre usuarios
7. **Agregar attachments** (archivos adjuntos)
8. **Implementar subtareas**

## 📚 Referencias

- [Express Documentation](https://expressjs.com/)
- [Prisma Documentation](https://www.prisma.io/docs)
- [JWT Best Practices](https://jwt.io/introduction)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

## ✅ Checklist

- [ ] Proyecto generado
- [ ] Dependencias instaladas
- [ ] Base de datos configurada
- [ ] Migraciones ejecutadas
- [ ] Tests pasando
- [ ] API corriendo en local
- [ ] Endpoints probados con Postman/curl
- [ ] Docker funcionando
- [ ] Listo para deploy

---

**¡Felicitaciones! Has creado tu primera API REST con Framework Elite Copilot.** 🎉

[← Volver a Ejemplos](README.md) | [Siguiente: PWA Todo →](02-pwa-todo.md)
