# 🛍️ DeliLocal

> Plataforma de pedidos en línea **mono-tienda** con seguimiento en tiempo real, catálogo interactivo, soporte multimoneda (USD/VES) y gestión operativa para comercios locales.

[![TypeScript](https://img.shields.io/badge/TypeScript-5.5+-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Turborepo](https://img.shields.io/badge/Turborepo-Monorepo-EF4444?logo=turborepo&logoColor=white)](https://turbo.build/repo)
[![NestJS](https://img.shields.io/badge/NestJS-10.x-E0234E?logo=nestjs&logoColor=white)](https://nestjs.com/)
[![Next.js](https://img.shields.io/badge/Next.js-14_(App_Router)-000000?logo=next.js&logoColor=white)](https://nextjs.org/)
[![Expo](https://img.shields.io/badge/Expo-SDK_51-000020?logo=expo&logoColor=white)](https://expo.dev/)
[![Socket.IO](https://img.shields.io/badge/Socket.IO-Realtime-010101?logo=socket.io&logoColor=white)](https://socket.io/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)

---

## 📖 Descripción del Proyecto

**DeliLocal** es una solución integral diseñada para negocios gastronómicos y comercios locales que operan bajo un modelo mono-tienda (un despliegue por negocio). Permite a los clientes explorar el catálogo y ordenar tanto desde la web como desde una aplicación móvil nativa, mientras que el equipo del negocio gestiona el flujo operativo en tiempo real desde un panel de control administrativo.

### 🇻🇪 Características adaptadas al mercado local
- **Doble moneda (USD / VES):** El dólar es la moneda base de almacenamiento y cálculo; los precios se convierten y presentan en bolívares (VES) en base a la tasa del día oficial (BCV) o ajuste manual administrativo.
- **Pagos manuales:** Soporte para métodos habituales de pago en Venezuela (Pago Móvil, Zelle, efectivo en entrega) con flujo de confirmación y conciliación manual por el administrador.
- **Seguimiento en tiempo real:** Comunicación bidireccional vía WebSockets (Socket.IO) para actualización instantánea del estado del pedido (Pendiente → Confirmado → En Preparación → Listo → Entregado).

---

## 🏗️ Arquitectura y Stack Tecnológico

El proyecto está organizado como un **monorepo estructurado con Turborepo y npm workspaces**:

| Capa | Tecnologías principales |
|---|---|
| **API Backend** | NestJS 10, Prisma ORM 5, PostgreSQL 16, Socket.IO, JWT + Refresh Tokens, bcrypt |
| **Frontend Web** | Next.js 14 (App Router, Server Components), Tailwind CSS, shadcn/ui, TanStack Query |
| **App Móvil** | React Native, Expo SDK 51, Expo Router, Zustand (persistencia con AsyncStorage) |
| **Lógica Compartida** | `@delilocal/shared` (tipos TypeScript, esquemas Zod, constantes de eventos, utilidades monetarias) |
| **Testing** | Vitest |
| **Almacenamiento** | Cloudinary (imágenes de productos y comprobantes) |

---

## 📁 Estructura del Monorepo

```text
delilocal/
├── apps/
│   ├── api/          # Servidor backend NestJS (módulos: auth, catalog, orders, payments, realtime, etc.)
│   ├── web/          # Aplicación web Next.js 14 (tienda de clientes + dashboard administrativo)
│   └── mobile/       # Aplicación móvil Expo / React Native
├── packages/
│   └── shared/       # Librería compartida (Zod schemas, tipos, máquina de estados, helpers)
├── Documentacion/    # Guías detalladas de arquitectura y diseño funcional
├── turbo.json        # Configuración de pipelines de compilación y ejecución
└── package.json      # Configuración de workspaces raíz
```

---

## ⚡ Reglas de Negocio Clave

1. **Precios garantizados por el servidor:** El cliente únicamente envía `productId` y `quantity`. El servidor consulta los precios vigentes en base de datos y calcula los totales para evitar cualquier manipulación en frontend.
2. **Snapshots inmutables:** Cada pedido almacena la tasa de cambio (`exchangeRate`), el total en VES y una copia histórica del nombre y precio de cada producto al momento exacto de la compra.
3. **Máquina de estados estricta:** Todas las transiciones de estado (`PENDING` → `CONFIRMED` → `PREPARING` → `READY` → `DELIVERED` o `CANCELLED`) están estrictamente validadas en el backend contra la matriz de transiciones permitidas.
4. **Propiedad de recursos:** Los clientes solo pueden consultar y recibir eventos en tiempo real de sus propios pedidos mediante validación de pertenencia e integración JWT en WebSockets.

---

## 🚀 Inicio Rápido

### 1. Prerrequisitos
- **Node.js:** `>= 20.0.0`
- **npm:** `>= 10.0.0`
- **Docker** (para levantar PostgreSQL fácilmente)

### 2. Clonar el repositorio e instalar dependencias
```bash
git clone https://github.com/rjmoreno2022/DeliLocal.git
cd DeliLocal
npm install
```

### 3. Configurar variables de entorno
Copia el archivo de ejemplo en la raíz y configura tus variables según tu entorno:
```bash
cp .env.example .env
```

### 4. Iniciar la Base de Datos con Docker
```bash
docker run --name delilocal-db -e POSTGRES_PASSWORD=dev -e POSTGRES_DB=delilocal -p 5432:5432 -d postgres:16
```

### 5. Ejecutar migraciones de Prisma
```bash
npx prisma migrate dev
npx prisma db seed
```

### 6. Iniciar el entorno de desarrollo
Levanta todas las aplicaciones concurrentemente a través de Turborepo:
```bash
npm run dev
```

---

## 🛠️ Comandos Disponibles

| Comando | Descripción |
|---|---|
| `npm run dev` | Inicia simultáneamente la API, el cliente Web y el dev server Móvil |
| `npm run build` | Compila todos los proyectos para producción |
| `npm run lint` | Ejecuta el análisis estático de código en todo el monorepo |
| `npm run typecheck` | Comprobación estricta de tipos de TypeScript |
| `npm run test` | Ejecuta las suites de pruebas con Vitest |

---

## 📚 Documentación Adicional

- [Documentación Técnica y Guía de Desarrollo](Documentacion/DeliLocal-Documentacion-Desarrollo-v1.0.md)
- [Reglas y Guía para Agentes de IA](AGENTS.md)
