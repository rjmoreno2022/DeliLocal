# 🚚 DeliLocal — Guía de Desarrollo v1.0

**Plataforma de pedidos con seguimiento en tiempo real para negocios locales.**
Web + Móvil + API — Documentación completa para desarrollo asistido por IA.

| | |
|---|---|
| ⏱ Duración | 14 semanas |
| 🛠 Fases | 8 |
| 📚 Nivel | Intermedio |
| 🌎 Lenguaje | TypeScript end-to-end |

**Stack:** TypeScript · NestJS · Next.js · React Native + Expo · PostgreSQL + Prisma · Socket.IO · Turborepo

---

## Tabla de Contenidos

1. [Introducción y Visión](#1-introducción-y-visión)
2. [Cómo Trabajar con un Modelo de IA](#2-cómo-trabajar-con-un-modelo-de-ia)
3. [Stack Tecnológico](#3-stack-tecnológico)
4. [Arquitectura de Software](#4-arquitectura-de-software)
5. [Estructura del Monorepo](#5-estructura-del-monorepo)
6. [Modelo de Dominio](#6-modelo-de-dominio)
7. [Base de Datos (Prisma)](#7-base-de-datos-prisma--postgresql)
8. [API REST (NestJS)](#8-api-rest-nestjs)
9. [WebSockets en Tiempo Real](#9-websockets-en-tiempo-real)
10. [Autenticación y Seguridad](#10-autenticación-y-seguridad)
11. [Frontend Web (Next.js)](#11-frontend-web-nextjs-14)
12. [App Móvil (React Native + Expo)](#12-app-móvil-react-native--expo)
13. [Doble Moneda (USD/VES)](#13-manejo-de-doble-moneda-usd--ves)
14. [Features del MVP](#14-features-del-mvp--priorización)
15. [Plan de Desarrollo por Fases](#15-plan-de-desarrollo-paso-a-paso)
16. [Mejores Prácticas](#16-mejores-prácticas-y-estándares)
17. [Testing](#17-testing)
18. [CI/CD y Deploy](#18-cicd-y-deploy)
19. [Comandos Rápidos](#19-comandos-de-referencia-rápida)
20. [Glosario](#20-glosario)
21. [Recursos](#21-recursos-y-referencias)

---

# 1. Introducción y Visión

## ¿Qué es DeliLocal?

DeliLocal es una **plataforma de pedidos en línea para un negocio local** (restaurante, bodega, farmacia, pastelería). El cliente navega el catálogo desde la app móvil o la web, arma su pedido, lo envía y **sigue su estado en tiempo real**. El negocio recibe el pedido al instante, lo gestiona desde un dashboard y actualiza su estado (recibido → preparando → listo → entregado).

## El problema que resuelve (contexto Venezuela)

- **Caos de WhatsApp:** los negocios reciben pedidos por chat, se pierden mensajes, no hay historial ni trazabilidad.
- **Precios en doble moneda:** los productos se cotizan en USD pero se pagan en bolívares; la tasa cambia a diario.
- **Métodos de pago locales:** pago móvil, Zelle, efectivo USD — ninguna pasarela internacional los soporta bien.
- **Sin seguimiento:** el cliente pregunta "¿cómo va mi pedido?" una y otra vez.

## Decisiones de producto (ya tomadas)

| Decisión | Opción elegida | Consecuencia técnica |
|---|---|---|
| Modelo de negocio | **Mono-tienda** (un negocio por despliegue) | No se requiere multi-tenancy; la tabla `Store` existe pero tiene una sola fila, lo que facilita escalar a multi-tienda después. |
| Plataformas | **Móvil y web para todo** | App móvil (Expo) + app web (Next.js) comparten la misma API. Lógica compartida en un paquete `shared`. |
| Tiempo real | **Sí, WebSockets** | NestJS Gateways con Socket.IO. Los clientes se suscriben a eventos de sus pedidos. |
| Pagos | **Confirmación manual** (pago móvil / Zelle / efectivo) | El cliente reporta la referencia de pago; el negocio la confirma. Sin pasarelas de pago en el MVP. |

## Roles de usuario

| Rol | Descripción | Pantallas principales |
|---|---|---|
| `CUSTOMER` | Cliente que hace pedidos | Catálogo, carrito, checkout, mis pedidos, seguimiento |
| `ADMIN` | Dueño del negocio | Dashboard, gestión de menú, gestión de pedidos, configuración |
| `STAFF` | Empleado que atiende pedidos | Cola de pedidos, actualización de estados |

## Objetivos de aprendizaje del proyecto

Este proyecto está diseñado para que domines las tecnologías con más demanda de trabajo remoto:

1. **TypeScript** en frontend, backend y móvil (un solo lenguaje, tres plataformas).
2. **React + Next.js**: el stack web más pedido del mercado.
3. **React Native + Expo**: móvil multiplataforma con un solo código.
4. **NestJS**: framework backend con arquitectura profesional (muy similar a ASP.NET, lo que aprovecha tu experiencia en .NET).
5. **WebSockets**: comunicación en tiempo real (muy valorado en entrevistas).
6. **Monorepos**: cómo las empresas modernas organizan proyectos grandes.
7. **CI/CD y deploy**: llevar el proyecto a producción real.

> 💡 **Conexión con FinTrack:** vienes de Clean Architecture en .NET. En esta guía verás constantes paralelismos: los *módulos* de NestJS ≈ tus proyectos de solution, la *inyección de dependencias* es idéntica en concepto, los *decoradores* ≈ atributos de C#, y Prisma ≈ Entity Framework. Este paralelismo acelerará tu aprendizaje.

---

# 2. Cómo Trabajar con un Modelo de IA

Esta guía está escrita para que puedas entregársela (completa o por secciones) a un modelo de IA y obtener código de calidad profesional. Sigue este flujo de trabajo:

## Reglas de oro

1. **Una fase a la vez.** No pidas "construye todo el proyecto". Pide la Fase 1, verifícala, y recién entonces pasa a la Fase 2.
2. **Contexto explícito.** Al iniciar cada sesión con la IA, pégale: la sección de Stack (3), la de Estructura (5) y la fase actual del Plan (15).
3. **Verifica antes de continuar.** Cada fase tiene un **entregable** verificable. No avances si no se cumple.
4. **Entiende lo que recibes.** Cada bloque *🎓 Aprende* de esta guía explica el porqué. Si la IA genera algo que no entiendes, pídele que lo explique línea por línea antes de aceptarlo.
5. **Commits por paso.** Haz commit después de cada paso completado y verificado, no al final de la fase.

## Plantilla de prompt para cada fase

```text
Estoy construyendo DeliLocal, una plataforma de pedidos para un negocio local.

STACK (no te desvíes de él):
- Monorepo con Turborepo + npm workspaces
- API: NestJS 10 + Prisma 5 + PostgreSQL 16
- Web: Next.js 14 (App Router) + Tailwind CSS + shadcn/ui
- Móvil: Expo SDK 51 (React Native) + Expo Router
- Tiempo real: Socket.IO (NestJS Gateways)
- Auth: JWT access + refresh tokens, bcrypt
- TODO el código en TypeScript estricto

CONTEXTO ACTUAL: [pega aquí lo que ya existe]

TAREA (Fase X, paso Y): [pega el paso exacto de la guía]

REGLAS:
- Sigue la estructura de carpetas definida en la guía
- Explica cada decisión no obvia con comentarios
- No agregues dependencias sin justificarlas
- Genera también los tests cuando el paso los pida
- Al final, dime qué comandos ejecutar para verificar que funciona
```

> ⚠️ **Error común:** dejar que la IA elija librerías alternativas ("podríamos usar X en lugar de Y"). Esto fragmenta el stack y genera inconsistencias entre fases. La respuesta correcta es: "No, usa el stack definido".

> 🎓 **Aprende: ¿por qué un documento como este?**
> Los modelos de IA no tienen memoria entre sesiones y tienden a improvisar arquitecturas distintas cada vez. Una guía como esta actúa como **fuente única de verdad** (single source of truth): define decisiones una vez y obliga a que todas las sesiones de desarrollo sean consistentes. Es el equivalente moderno del "documento de diseño" que los equipos senior escriben antes de codear.

---

# 3. Stack Tecnológico

## Resumen

| Capa | Tecnología | Versión | Equivalente en tu experiencia .NET |
|---|---|---|---|
| Lenguaje | **TypeScript** | 5.x | C# (tipado estático, clases, interfaces) |
| Backend | **NestJS** | 10.x | ASP.NET Core (DI, decoradores, middleware) |
| ORM | **Prisma** | 5.x | Entity Framework Core |
| Base de datos | **PostgreSQL** | 16 | SQL Server |
| Web | **Next.js + React** | 14.x / 18.x | Blazor (componentes con estado) |
| Estilos web | **Tailwind CSS + shadcn/ui** | 3.x | Bootstrap / MudBlazor |
| Móvil | **Expo (React Native)** | SDK 51 | .NET MAUI |
| Tiempo real | **Socket.IO** | 4.x | SignalR |
| Monorepo | **Turborepo** | 2.x | Una Solution con múltiples proyectos |
| Validación | **Zod** | 3.x | FluentValidation / DataAnnotations |
| Testing | **Vitest + Testing Library** | 1.x | xUnit + Moq |
| Imágenes | **Cloudinary** | — | Azure Blob Storage |
| Deploy | **Vercel + Railway + EAS** | — | Azure App Service |

## ¿Por qué estas elecciones?

### TypeScript en todo
Un solo lenguaje reduce la carga cognitiva y permite **compartir tipos entre frontend y backend**: si cambias un campo en la API, TypeScript te avisa en la web y en el móvil al instante. Esto es imposible con stacks mixtos.

### NestJS (no Express puro)
Express es minimalista: no impone arquitectura y los proyectos terminan desordenados. NestJS impone módulos, controladores, servicios e inyección de dependencias — la misma estructura que ya conoces de ASP.NET. Además, es de los frameworks de Node más demandados laboralmente.

### Prisma (no TypeORM)
Prisma define el modelo en un archivo `schema.prisma` declarativo y genera un cliente 100% tipado. Sus migraciones son predecibles y su autocompletado es superior. TypeORM está en declive; Prisma domina el mercado actual.

### Next.js App Router (no Create React App)
CRA está muerto oficialmente. Next.js es el estándar: ofrece Server Components, rutas por archivos, SSR/SSG y deploy trivial en Vercel. Es lo que aparece en la gran mayoría de ofertas de trabajo remoto de React.

### Expo (no React Native "bare")
Expo elimina la configuración nativa (Xcode/Android Studio) para el 95% de los casos, añade un ecosistema de librerías probadas (notificaciones, cámara, etc.) y permite builds en la nube con EAS. Para una app como esta, no necesitas nada más.

> 🎓 **Aprende: ¿qué es un monorepo?**
> Un **monorepo** es un único repositorio Git que contiene varios proyectos relacionados (API, web, móvil) más paquetes compartidos. La alternativa (un repo por proyecto) obliga a versionar y sincronizar cambios entre repos. En el monorepo, un cambio en un tipo compartido se refleja inmediatamente en las tres apps. Empresas como Google, Meta y Vercel usan monorepos. **Turborepo** orquesta las tareas (build, test, lint) cacheando resultados para que todo sea rápido.

---

# 4. Arquitectura de Software

## Vista general del sistema

```text
                        ┌─────────────────────────────┐
                        │         CLIENTES            │
                        │                             │
                        │   ┌─────────┐  ┌─────────┐  │
                        │   │ Web App │  │ App     │  │
                        │   │ Next.js │  │ Expo/RN │  │
                        │   └────┬────┘  └────┬────┘  │
                        └────────┼────────────┼───────┘
                                 │            │
                    HTTPS (REST) │            │ WSS (Socket.IO)
                                 │            │
                        ┌────────▼────────────▼───────┐
                        │      API NestJS             │
                        │  ┌───────────────────────┐  │
                        │  │ Controllers (REST)    │  │
                        │  │ Gateways (WebSocket)  │  │
                        │  │ Services (lógica)     │  │
                        │  │ Guards + Interceptors │  │
                        │  └───────────┬───────────┘  │
                        │       Prisma │ ORM          │
                        └──────────────┼──────────────┘
                                       │
                        ┌──────────────▼──────────────┐
                        │      PostgreSQL 16          │
                        └─────────────────────────────┘

  Servicios externos: Cloudinary (imágenes) · Expo Push (notificaciones) · API tasa BCV
```

## Arquitectura interna de la API (NestJS)

NestJS organiza el código en **módulos por dominio**. Cada módulo encapsula sus controladores, servicios y DTOs. Es el equivalente a organizar tu solution de .NET por features en lugar de por capas técnicas.

```text
AppModule
├── AuthModule          → login, registro, refresh, guards JWT, roles
├── UsersModule         → perfil, direcciones del cliente
├── CatalogModule       → categorías y productos (el menú)
├── OrdersModule        → crear pedido, cambiar estado, historial  ← núcleo
├── PaymentsModule      → reportar referencia de pago, confirmar
├── NotificationsModule → push tokens, envío de notificaciones
├── ExchangeRateModule  → tasa USD/VES del día
├── RealtimeModule      → gateway Socket.IO, rooms por pedido
└── ConfigModule        → variables de entorno validadas
```

## Flujo de un pedido (el flujo más importante del sistema)

1. **Cliente crea pedido** → `POST /orders` con items y método de pago. La API valida stock y precios, calcula totales (USD y VES según la tasa del día) y guarda con estado `PENDING`.
2. **API emite evento WebSocket** `order:created` a la sala `store:admins` → el dashboard del negocio muestra el pedido nuevo al instante (con sonido) y se envía push notification.
3. **Negocio acepta** → `PATCH /orders/:id/status` a `CONFIRMED` → evento `order:updated` a la sala `order:{id}` → el cliente ve el cambio en vivo + push.
4. **Cliente reporta pago** → `POST /orders/:id/payment` con referencia (pago móvil/Zelle) → el negocio la confirma → estado de pago `VERIFIED`.
5. **Estados siguientes:** `PREPARING` → `READY` → `DELIVERED` (o `CANCELLED` con motivo). Cada cambio emite WebSocket + push.

> 🎓 **Aprende: REST vs WebSockets — ¿cuándo usar cada uno?**
> **REST** es request/response: el cliente pregunta, el servidor responde. Ideal para acciones (crear pedido, listar productos). **WebSockets** mantiene una conexión abierta bidireccional: el servidor puede "empujar" datos sin que el cliente pregunte. Ideal para eventos (tu pedido cambió de estado). Regla práctica: *comandos por REST, notificaciones por WebSocket*. Nunca uses WebSockets para CRUD ni REST con polling para tiempo real.

---

# 5. Estructura del Monorepo

```text
delilocal/
├── apps/
│   ├── api/                          # NestJS — backend
│   │   ├── src/
│   │   │   ├── auth/            # controller, service, guards, strategies, dto/
│   │   │   ├── users/           # perfil, direcciones
│   │   │   ├── catalog/         # categories, products
│   │   │   ├── orders/          # núcleo del negocio
│   │   │   ├── payments/        # referencias de pago
│   │   │   ├── notifications/   # expo push
│   │   │   ├── exchange-rate/   # tasa BCV
│   │   │   ├── realtime/        # websocket gateway
│   │   │   ├── prisma/          # prisma.service.ts
│   │   │   ├── app.module.ts
│   │   │   └── main.ts
│   │   ├── prisma/
│   │   │   ├── schema.prisma
│   │   │   └── seed.ts
│   │   └── package.json
│   │
│   ├── web/                          # Next.js 14 (App Router)
│   │   ├── app/
│   │   │   ├── (shop)/          # catálogo, carrito, checkout, pedidos (cliente)
│   │   │   ├── (admin)/admin/   # dashboard del negocio (protegido)
│   │   │   ├── (auth)/          # login, registro
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   ├── components/          # ui/ (shadcn) + componentes de negocio
│   │   ├── lib/                 # api-client, socket-client, hooks
│   │   └── package.json
│   │
│   └── mobile/                       # Expo + Expo Router
│       ├── app/                 # rutas (file-based, igual que Next)
│       │   ├── (tabs)/          # catálogo, carrito, pedidos, perfil
│       │   ├── order/[id].tsx   # seguimiento en vivo
│       │   └── _layout.tsx
│       ├── components/
│       ├── lib/                 # api-client, socket-client, storage
│       └── package.json
│
├── packages/
│   └── shared/                       # código compartido por las 3 apps
│       ├── src/
│       │   ├── types/           # Order, Product, User, enums (OrderStatus...)
│       │   ├── schemas/         # validaciones Zod (CreateOrderSchema...)
│       │   ├── constants/       # eventos WS, endpoints, límites
│       │   └── utils/           # formatMoney, currency math
│       └── package.json         # nombre: @delilocal/shared
│
├── turbo.json
├── package.json                     # workspaces + scripts raíz
├── .env.example
└── README.md
```

> 🎓 **Aprende: el paquete `shared` es la pieza más valiosa**
> El enum `OrderStatus` lo usan la API (para validar transiciones), la web (para mostrar badges de colores) y el móvil (para la línea de tiempo del pedido). Si viviera duplicado en tres lugares, tarde o temprano se desincronizaría. Al vivir en `@delilocal/shared`, hay **una sola definición** y TypeScript garantiza consistencia en las tres apps. Lo mismo aplica a los esquemas Zod: la misma validación corre en el servidor (seguridad) y en los formularios (UX).

> ℹ️ **Regla de dependencias:** `apps/*` pueden importar de `packages/*`. Los paquetes **nunca** importan de las apps. Las apps nunca importan entre sí. Esta dirección única evita dependencias circulares.

---

# 6. Modelo de Dominio

## Diagrama de entidades

```text
  ┌──────────┐ 1   N ┌───────────┐ N   1 ┌──────────────┐
  │   User   │───────│   Order   │───────│  OrderItem   │
  └────┬─────┘       └─────┬─────┘       └──────┬───────┘
       │ 1                 │ 1                  │ N
       │                   │                    │
       │ N                 │ 1                  │ N   1 ┌───────────┐
  ┌────▼─────┐       ┌─────▼──────┐            └───────│  Product  │
  │ Address  │       │  Payment   │                    └─────┬─────┘
  └──────────┘       └────────────┘                          │ N
                                                             │
  ┌──────────┐ 1   N ┌───────────┐                    N    1 │
  │  Store   │───────│ Category  │────────────────────────────┘
  └────┬─────┘       └───────────┘
       │ 1
       │ N
  ┌────▼─────────┐     ┌─────────────────┐
  │ ExchangeRate │     │  PushToken      │ (de User)
  └──────────────┘     └─────────────────┘
```

## Descripción de entidades

| Entidad | Responsabilidad | Campos clave |
|---|---|---|
| `Store` | El negocio (una sola fila en el MVP) | name, slug, logoUrl, whatsapp, isOpen, deliveryFeeUsd |
| `User` | Cualquier usuario del sistema | email, passwordHash, name, phone, role (CUSTOMER/ADMIN/STAFF) |
| `Address` | Direcciones de entrega del cliente | label, street, reference, lat/lng (opcional) |
| `Category` | Agrupa productos del menú | name, sortOrder, isActive |
| `Product` | Ítem del menú | name, description, priceUsd, imageUrl, isAvailable, categoryId |
| `Order` | El pedido — entidad central | orderNumber, status, subtotalUsd, deliveryFeeUsd, totalUsd, exchangeRate, totalVes, paymentMethod, notes |
| `OrderItem` | Línea del pedido (snapshot del producto) | productName, unitPriceUsd, quantity, notes |
| `Payment` | Reporte de pago del cliente | method, reference, amountVes/amountUsd, status (REPORTED/VERIFIED/REJECTED) |
| `ExchangeRate` | Tasa USD/VES por día | rate, source, effectiveDate |
| `PushToken` | Token Expo del dispositivo del usuario | token, platform |

## Enum: ciclo de vida del pedido

```typescript
// packages/shared/src/types/order.ts
export enum OrderStatus {
  PENDING   = 'PENDING',    // creado por el cliente, esperando aceptación del negocio
  CONFIRMED = 'CONFIRMED',  // negocio aceptó el pedido
  PREPARING = 'PREPARING',  // en cocina / en preparación
  READY     = 'READY',      // listo para entrega o retiro
  DELIVERED = 'DELIVERED',  // entregado (estado final)
  CANCELLED = 'CANCELLED',  // cancelado (estado final)
}

// Máquina de estados: qué transiciones son legales
export const ORDER_TRANSITIONS: Record<OrderStatus, OrderStatus[]> = {
  [OrderStatus.PENDING]:   [OrderStatus.CONFIRMED, OrderStatus.CANCELLED],
  [OrderStatus.CONFIRMED]: [OrderStatus.PREPARING, OrderStatus.CANCELLED],
  [OrderStatus.PREPARING]: [OrderStatus.READY, OrderStatus.CANCELLED],
  [OrderStatus.READY]:     [OrderStatus.DELIVERED],
  [OrderStatus.DELIVERED]: [],  // final
  [OrderStatus.CANCELLED]: [],  // final
};
```

> 🎓 **Aprende: ¿por qué una máquina de estados explícita?**
> Sin ella, cualquier endpoint podría poner un pedido `DELIVERED` directamente desde `PENDING`, o "resucitar" un pedido cancelado. Al declarar las transiciones legales en un solo lugar (y compartirlo vía `@delilocal/shared`), el backend valida cada cambio y el frontend puede *deshabilitar botones imposibles* usando la misma tabla. Es el patrón **State Machine**, y es una pregunta clásica de entrevistas técnicas.

## Reglas de negocio innegociables

1. **Snapshot de precios:** `OrderItem` copia el nombre y precio del producto en el momento de la compra. Si mañana el producto cambia de precio, los pedidos históricos no se alteran.
2. **Snapshot de tasa:** `Order` guarda la `exchangeRate` usada. El total en VES del pedido no cambia si la tasa cambia después.
3. **Stock implícito:** un producto con `isAvailable = false` no puede añadirse a pedidos nuevos (validado en servidor al crear la orden, no solo en la UI).
4. **Número de orden legible:** `orderNumber` es secuencial por día (ej. `#20250612-0042`) para que el negocio y el cliente hablen de "el pedido 42" y no de un UUID.
5. **Cancelación con motivo:** todo pedido cancelado registra `cancelReason` y quién lo canceló.

---

# 7. Base de Datos (Prisma + PostgreSQL)

## Schema completo

Este es el corazón de los datos del sistema. Prisma genera a partir de aquí las migraciones SQL y un cliente TypeScript totalmente tipado.

```prisma
// apps/api/prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum Role {
  CUSTOMER
  ADMIN
  STAFF
}

enum OrderStatus {
  PENDING
  CONFIRMED
  PREPARING
  READY
  DELIVERED
  CANCELLED
}

enum PaymentMethod {
  CASH_USD        // efectivo dólares
  CASH_VES        // efectivo bolívares
  PAGO_MOVIL      // pago móvil (Venezuela)
  ZELLE
  BINANCE
}

enum PaymentStatus {
  REPORTED   // cliente reportó la referencia
  VERIFIED   // negocio confirmó recepción
  REJECTED   // negocio rechazó (referencia inválida)
}

model Store {
  id             String   @id @default(cuid())
  name           String
  slug           String   @unique
  logoUrl        String?
  whatsapp       String?
  address        String?
  isOpen         Boolean  @default(true)
  deliveryFeeUsd Decimal  @db.Decimal(10, 2) @default(0)
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt
  categories     Category[]
  products       Product[]
  orders         Order[]
}

model User {
  id           String      @id @default(cuid())
  email        String      @unique
  passwordHash String
  name         String
  phone        String?
  role         Role        @default(CUSTOMER)
  createdAt    DateTime    @default(now())
  updatedAt    DateTime    @updatedAt
  addresses    Address[]
  orders       Order[]
  pushTokens   PushToken[]
}

model Address {
  id         String  @id @default(cuid())
  userId     String
  label      String  // "Casa", "Oficina"
  street     String
  reference  String? // "frente a la farmacia, portón verde"
  isDefault  Boolean @default(false)
  user       User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
}

model Category {
  id        String    @id @default(cuid())
  storeId   String
  name      String
  sortOrder Int       @default(0)
  isActive  Boolean   @default(true)
  store     Store     @relation(fields: [storeId], references: [id], onDelete: Cascade)
  products  Product[]

  @@index([storeId])
}

model Product {
  id          String      @id @default(cuid())
  storeId     String
  categoryId  String
  name        String
  description String?
  priceUsd    Decimal     @db.Decimal(10, 2)
  imageUrl    String?
  isAvailable Boolean     @default(true)
  createdAt   DateTime    @default(now())
  updatedAt   DateTime    @updatedAt
  store       Store       @relation(fields: [storeId], references: [id], onDelete: Cascade)
  category    Category    @relation(fields: [categoryId], references: [id])
  orderItems  OrderItem[]

  @@index([storeId, isAvailable])
  @@index([categoryId])
}

model Order {
  id            String        @id @default(cuid())
  orderNumber   String        @unique   // "20250612-0042"
  storeId       String
  customerId    String
  addressId     String?
  status        OrderStatus   @default(PENDING)
  paymentMethod PaymentMethod
  subtotalUsd   Decimal       @db.Decimal(10, 2)
  deliveryFeeUsd Decimal      @db.Decimal(10, 2)
  totalUsd      Decimal       @db.Decimal(10, 2)
  exchangeRate  Decimal       @db.Decimal(12, 2) // snapshot de la tasa
  totalVes      Decimal       @db.Decimal(14, 2)
  notes         String?
  cancelReason  String?
  createdAt     DateTime      @default(now())
  updatedAt     DateTime      @updatedAt
  store         Store         @relation(fields: [storeId], references: [id])
  customer      User          @relation(fields: [customerId], references: [id])
  items         OrderItem[]
  payment       Payment?

  @@index([customerId, createdAt])
  @@index([storeId, status])
}

model OrderItem {
  id           String  @id @default(cuid())
  orderId      String
  productId    String?
  productName  String  // snapshot: no depende del producto actual
  unitPriceUsd Decimal @db.Decimal(10, 2)
  quantity     Int
  notes        String? // "sin cebolla"
  order        Order   @relation(fields: [orderId], references: [id], onDelete: Cascade)
  product      Product? @relation(fields: [productId], references: [id], onDelete: SetNull)

  @@index([orderId])
}

model Payment {
  id         String        @id @default(cuid())
  orderId    String        @unique
  method     PaymentMethod
  reference  String?       // nro. de referencia del pago móvil / Zelle
  amountVes  Decimal?      @db.Decimal(14, 2)
  amountUsd  Decimal?      @db.Decimal(10, 2)
  status     PaymentStatus @default(REPORTED)
  verifiedBy String?       // userId del admin/staff que confirmó
  createdAt  DateTime      @default(now())
  updatedAt  DateTime      @updatedAt
  order      Order         @relation(fields: [orderId], references: [id], onDelete: Cascade)
}

model ExchangeRate {
  id            String   @id @default(cuid())
  rate          Decimal  @db.Decimal(12, 2) // VES por 1 USD
  source        String   @default("BCV")
  effectiveDate DateTime @db.Date
  createdAt     DateTime @default(now())

  @@unique([source, effectiveDate])
}

model PushToken {
  id        String   @id @default(cuid())
  userId    String
  token     String   @unique // ExponentPushToken[xxxx]
  platform  String   // "ios" | "android"
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
}
```

## Decisiones de diseño explicadas

| Decisión | Por qué |
|---|---|
| `Decimal` para dinero, nunca `Float` | Los flotantes tienen errores de redondeo binario (`0.1 + 0.2 ≠ 0.3`). En dinero es inaceptable. PostgreSQL `DECIMAL` es exacto. |
| `onDelete: Cascade` en items, `SetNull` en producto del item | Si se borra un pedido, sus líneas mueren con él. Pero si se borra un *producto* del catálogo, el pedido histórico conserva su línea (con el snapshot de nombre/precio) y solo pierde el enlace. |
| Índices en `(storeId, status)` y `(customerId, createdAt)` | Son las consultas calientes: "pedidos activos del negocio" (dashboard) e "historial del cliente". Sin índices, esas queries escanean toda la tabla. |
| `cuid()` en vez de autoincremental | IDs no predecibles (seguridad: nadie adivina `/orders/105`) y generables sin consultar la BD. |
| `updatedAt @updatedAt` | Prisma lo actualiza solo. Útil para auditoría y para ordenar la cola del dashboard. |

## Seed data

El archivo `prisma/seed.ts` debe crear: la tienda única, un usuario admin (`admin@delilocal.com`), un cliente de prueba, 3-4 categorías, 8-12 productos con imágenes y la tasa del día. Se ejecuta con `npx prisma db seed`.

> ✅ **Verificación:** tras correr migraciones y seed, abre `npx prisma studio` — verás todas las tablas con datos navegables en el navegador. Es el "SQL Server Management Studio" de Prisma.

---

# 8. API REST (NestJS)

## Convenciones de la API

- **Base URL:** `/api/v1` (versionado desde el día uno).
- **Respuestas:** JSON. Errores con formato uniforme `{ statusCode, message, error, timestamp, path }` (NestJS lo da gratis con filtros de excepción).
- **Validación:** todo DTO se valida con `class-validator` + `ValidationPipe` global con `whitelist: true` (rechaza campos no declarados).
- **Documentación:** Swagger autogenerado en `/api/docs` con `@nestjs/swagger`.
- **Paginación:** listas con `?page=1&limit=20`, respuesta `{ data: [], total, page, totalPages }`.

## Mapa de endpoints

| Método | Endpoint | Rol | Descripción |
|---|---|---|---|
| POST | `/auth/register` | Público | Registro de cliente |
| POST | `/auth/login` | Público | Login → access + refresh token |
| POST | `/auth/refresh` | Público | Renovar access token |
| GET | `/users/me` | Todos | Perfil propio |
| POST | `/users/me/addresses` | Customer | Crear dirección |
| GET | `/catalog/categories` | Público | Categorías con productos disponibles |
| POST | `/catalog/products` | Admin | Crear producto (imagen → Cloudinary) |
| PATCH | `/catalog/products/:id` | Admin | Editar precio, disponibilidad, etc. |
| POST | `/orders` | Customer | Crear pedido (valida items, calcula totales) |
| GET | `/orders/my` | Customer | Historial propio (paginado) |
| GET | `/orders/:id` | Dueño/Admin | Detalle del pedido |
| GET | `/orders?status=` | Admin/Staff | Cola de pedidos del negocio |
| PATCH | `/orders/:id/status` | Admin/Staff | Cambiar estado (valida máquina de estados) |
| POST | `/orders/:id/payment` | Customer | Reportar referencia de pago |
| PATCH | `/orders/:id/payment/verify` | Admin | Verificar o rechazar pago |
| GET | `/exchange-rate/today` | Público | Tasa USD/VES actual |
| POST | `/notifications/push-token` | Todos | Registrar token del dispositivo |

## Ejemplo: el endpoint más importante (crear pedido)

```typescript
// apps/api/src/orders/orders.service.ts (extracto)
@Injectable()
export class OrdersService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly realtime: RealtimeGateway,
    private readonly rates: ExchangeRateService,
  ) {}

  async create(customerId: string, dto: CreateOrderDto) {
    // 1. Traer productos reales de la BD — NUNCA confiar en precios del cliente
    const products = await this.prisma.product.findMany({
      where: { id: { in: dto.items.map(i => i.productId) }, isAvailable: true },
    });
    if (products.length !== dto.items.length)
      throw new BadRequestException('Hay productos no disponibles');

    // 2. Calcular totales en servidor con precios de la BD
    const subtotal = dto.items.reduce((sum, item) => {
      const p = products.find(p => p.id === item.productId)!;
      return sum + Number(p.priceUsd) * item.quantity;
    }, 0);

    const rate = await this.rates.getTodayRate();
    const store = await this.prisma.store.findFirstOrThrow();
    const total = subtotal + Number(store.deliveryFeeUsd);

    // 3. Persistir pedido + items + snapshots en una transacción
    const order = await this.prisma.order.create({
      data: {
        orderNumber: await this.generateOrderNumber(),
        storeId: store.id, customerId,
        status: 'PENDING',
        paymentMethod: dto.paymentMethod,
        subtotalUsd: subtotal,
        deliveryFeeUsd: store.deliveryFeeUsd,
        totalUsd: total,
        exchangeRate: rate,
        totalVes: total * rate,
        notes: dto.notes, addressId: dto.addressId,
        items: { create: dto.items.map(i => {
          const p = products.find(p => p.id === i.productId)!;
          return { productId: p.id, productName: p.name,
                   unitPriceUsd: p.priceUsd, quantity: i.quantity, notes: i.notes };
        })},
      },
      include: { items: true },
    });

    // 4. Notificar al negocio en tiempo real
    this.realtime.emitToAdmins('order:created', order);
    return order;
  }
}
```

> 🚨 **Regla de seguridad #1 de cualquier e-commerce:** los precios *jamás* se toman del request del cliente. El cliente envía solo `productId` y `quantity`; el servidor busca el precio real en la base de datos. De lo contrario, alguien puede editar el request y pedirse todo a $0.01.

> 🎓 **Aprende: la anatomía de un módulo NestJS**
> **Controller** = recibe HTTP, valida el DTO, delega (≈ tu Controller de ASP.NET). **Service** = lógica de negocio, inyectado por constructor (≈ tus servicios de Application). **Module** = declara qué controllers y providers existen y qué exporta a otros módulos (≈ registrar servicios en `Program.cs`). **DTO** = clase con decoradores `@IsString()`, `@Min(1)` que NestJS valida automáticamente (≈ DataAnnotations). **Guard** = decide si el request puede pasar (≈ middleware de autorización / policies).

---

# 9. WebSockets en Tiempo Real

## Diseño del canal

Usamos **Socket.IO** (no WebSocket crudo) porque añade reconexión automática, salas (rooms) y fallback a polling — gratis.

| Concepto | Valor |
|---|---|
| Namespace | `/realtime` |
| Autenticación | JWT enviado en el handshake (`auth: { token }`), verificado por un Guard de WS |
| Room `store:admins` | Todos los ADMIN/STAFF conectados entran aquí al conectar |
| Room `order:{orderId}` | El cliente dueño del pedido se une para seguimiento en vivo |

## Eventos (servidor → cliente)

| Evento | Room destino | Payload | Cuándo |
|---|---|---|---|
| `order:created` | `store:admins` | Pedido completo | Cliente crea pedido |
| `order:updated` | `order:{id}` + `store:admins` | `{ orderId, status, updatedAt }` | Cambio de estado |
| `payment:reported` | `store:admins` | `{ orderId, method, reference }` | Cliente reporta pago |
| `store:toggled` | todos | `{ isOpen }` | Negocio abre/cierra |

## Gateway de NestJS

```typescript
// apps/api/src/realtime/realtime.gateway.ts
@WebSocketGateway({ namespace: '/realtime', cors: { origin: true } })
export class RealtimeGateway implements OnGatewayConnection {
  @WebSocketServer() server: Server;

  constructor(private readonly jwt: JwtService) {}

  // Se ejecuta en cada conexión: autenticar y unir a rooms
  async handleConnection(socket: Socket) {
    try {
      const payload = await this.jwt.verifyAsync(socket.handshake.auth.token);
      socket.data.user = payload;  // { sub, role }

      if (payload.role === 'ADMIN' || payload.role === 'STAFF')
        socket.join('store:admins');
    } catch {
      socket.disconnect();  // sin JWT válido no hay canal
    }
  }

  // El cliente llama esto para seguir un pedido suyo
  @SubscribeMessage('order:watch')
  async watchOrder(@ConnectedSocket() socket: Socket,
                   @MessageBody() orderId: string) {
    // TODO: verificar que socket.data.user.sub es dueño del pedido (o admin)
    socket.join(`order:${orderId}`);
  }

  emitToAdmins(event: string, data: unknown) {
    this.server.to('store:admins').emit(event, data);
  }

  emitToOrder(orderId: string, event: string, data: unknown) {
    this.server.to(`order:${orderId}`).emit(event, data);
  }
}
```

> 🎓 **Aprende: ¿qué es una "room"?**
> Una room es un canal con nombre dentro de la conexión WebSocket. En lugar de enviar un evento a *todos* los conectados (`server.emit`), lo envías solo a quienes se unieron a esa sala (`server.to('order:abc').emit`). Es lo mismo que los **grupos de SignalR** que quizás usaste en Blazor. Sin rooms, cada cliente recibiría los pedidos de todos los demás — un problema de privacidad y de tráfico.

> ⚠️ **Importante:** los nombres de eventos (`order:created`, etc.) deben definirse como constantes en `packages/shared/src/constants/ws-events.ts` e importarse en API, web y móvil. Un typo en un string de evento es un bug silencioso muy difícil de encontrar.

---

# 10. Autenticación y Seguridad

## Esquema de tokens

| Token | Duración | Dónde se guarda | Propósito |
|---|---|---|---|
| **Access token** (JWT) | 15 min | Memoria (web) / SecureStore (móvil) | Autenticar cada request REST y el handshake WS |
| **Refresh token** (JWT) | 30 días | HttpOnly cookie (web) / SecureStore (móvil) | Obtener access tokens nuevos sin re-login |

## Reglas de seguridad del proyecto

1. **Contraseñas con bcrypt** (cost factor 12). Nunca en texto plano, nunca en logs, nunca en respuestas de la API.
2. **Autorización por roles con Guards:** decorador `@Roles('ADMIN')` + `RolesGuard` que lee el rol del JWT. El rol *siempre* se lee del token firmado, nunca del body.
3. **Propiedad de recursos:** un CUSTOMER solo ve sus propios pedidos. Verificar `order.customerId === user.sub` en el service, además del guard.
4. **Rate limiting:** `@nestjs/throttler` — máx. 10 req/min en `/auth/login` (anti fuerza bruta), 100 req/min global.
5. **Helmet + CORS estricto:** helmet activado; CORS solo para los orígenes de la web (y Expo en dev).
6. **Validación total de entrada:** `ValidationPipe` global con `whitelist: true` y `forbidNonWhitelisted: true`.
7. **Secretos en variables de entorno:** validados al arranque con `class-validator` en `config/env.validation.ts`. Si falta `JWT_SECRET`, la app no arranca (fail fast).
8. **SQL injection:** imposible usando Prisma (queries parametrizadas). Prohibido `$queryRawUnsafe` con interpolación de strings.

> 🎓 **Aprende: ¿por qué dos tokens y no uno solo?**
> Un access token corto (15 min) limita el daño si es robado: el atacante solo lo usa un rato. El refresh token largo permite no pedir login cada 15 minutos. Si el refresh se compromete, se puede revocar en servidor. Este patrón (**access + refresh**) es el estándar de la industria y casi seguro te lo preguntarán en entrevistas.

---

# 11. Frontend Web (Next.js 14)

## Mapa de rutas

| Ruta | Público | Descripción |
|---|---|---|
| `/` | Cliente | Catálogo con categorías y buscador |
| `/cart` | Cliente | Carrito (persiste en localStorage) |
| `/checkout` | Cliente (auth) | Dirección, método de pago, notas, resumen USD/VES |
| `/orders` | Cliente (auth) | Historial de pedidos |
| `/orders/[id]` | Cliente (auth) | Seguimiento en vivo (timeline + WebSocket) |
| `/login`, `/register` | Público | Autenticación |
| `/admin` | Admin/Staff | Dashboard: cola de pedidos en tiempo real |
| `/admin/menu` | Admin | CRUD de productos y categorías |
| `/admin/settings` | Admin | Datos del negocio, tasa manual, abrir/cerrar tienda |

## Decisiones técnicas web

- **Server Components por defecto:** el catálogo se renderiza en servidor (rápido, SEO). Solo son `'use client'` los componentes interactivos (carrito, botones, seguimiento en vivo).
- **Estado del carrito:** `Zustand` + persistencia en localStorage. No uses Redux para esto — es excesivo.
- **Data fetching:** `fetch` nativo en Server Components; `TanStack Query` en componentes cliente que necesitan revalidación (pedidos, dashboard).
- **Formularios:** `react-hook-form` + `zod` con los schemas de `@delilocal/shared`.
- **UI:** componentes de `shadcn/ui` (Button, Card, Dialog, Badge, Sheet, Toast) personalizados con la paleta del negocio.

## El dashboard en tiempo real (pieza estrella)

```tsx
// apps/web/app/(admin)/admin/page.tsx (extracto)
'use client';
export default function AdminDashboard() {
  const { data: orders } = useQuery({
    queryKey: ['orders', 'active'],
    queryFn: () => api.getActiveOrders(),
  });

  useEffect(() => {
    const socket = connectSocket();  // lib/socket-client.ts (con JWT)

    socket.on(WS_EVENTS.ORDER_CREATED, (order) => {
      playNotificationSound();       // 🔔 clave para el negocio
      queryClient.invalidateQueries(['orders']);
      toast.info(`¡Nuevo pedido #${order.orderNumber}!`);
    });

    socket.on(WS_EVENTS.ORDER_UPDATED, () =>
      queryClient.invalidateQueries(['orders']));

    return () => socket.disconnect();
  }, []);

  // Render: columnas por estado (kanban simple)
  // PENDING → CONFIRMED → PREPARING → READY, cada tarjeta con botón
  // para avanzar al siguiente estado legal (según ORDER_TRANSITIONS)
}
```

> 🎓 **Aprende: Server vs Client Components**
> En el App Router, **todo componente es de servidor por defecto**: se ejecuta en el servidor y manda HTML al navegador (cero JavaScript enviado, carga rapidísima). Cuando necesitas interactividad (`useState`, `useEffect`, onClick), marcas el componente con `'use client'` y React lo "hidrata" en el navegador. La regla práctica: *empuja los 'use client' lo más abajo posible en el árbol* — la página es server, solo el botón es client.

## Checklist de UX web

- Precios siempre en **doble moneda**: `$5.00 · Bs. 542,50` (tasa visible en el header).
- Estados de carga con **skeletons** (no spinners genéricos).
- Estados vacíos diseñados ("Aún no tienes pedidos" + botón al catálogo).
- Errores de API traducidos a mensajes humanos en español.
- Responsive primero móvil: el 80% de los clientes entrará desde el teléfono.

---

# 12. App Móvil (React Native + Expo)

## Navegación (Expo Router)

```text
app/
├── _layout.tsx          # providers (QueryClient, Auth, Socket)
├── index.tsx            # redirige según auth → /(tabs) o /login
├── (auth)/
│   ├── login.tsx
│   └── register.tsx
├── (tabs)/              # bottom tabs del cliente
│   ├── index.tsx        # catálogo
│   ├── cart.tsx         # carrito
│   ├── orders.tsx       # mis pedidos
│   └── profile.tsx      # perfil, direcciones, logout
├── order/
│   └── [id].tsx         # seguimiento en vivo del pedido
├── checkout.tsx         # modal de checkout
└── (admin)/             # solo visible si role ≠ CUSTOMER
    └── orders.tsx       # cola de pedidos para staff
```

## Librerías móviles clave

| Librería | Para qué |
|---|---|
| `expo-router` | Navegación por archivos (como Next.js — ya la conocerás) |
| `expo-notifications` | Push notifications + permiso + token del dispositivo |
| `expo-secure-store` | Guardar JWT de forma cifrada (Keychain/Keystore) |
| `@tanstack/react-query` | Data fetching con caché (misma librería que en web) |
| `zustand` | Carrito (misma librería que en web) |
| `socket.io-client` | Tiempo real (mismo cliente que en web) |
| `react-hook-form + zod` | Formularios con los schemas compartidos |

## Push notifications (flujo completo)

1. Al iniciar sesión, la app pide permiso y obtiene el `ExpoPushToken` del dispositivo.
2. Lo envía a `POST /notifications/push-token` → se guarda en la tabla `PushToken` asociado al usuario.
3. Cuando el pedido cambia de estado, el backend busca los tokens del cliente y envía la notificación vía la API de Expo (`expo-server-sdk`).
4. Al tocar la notificación, la app abre directamente `/order/[id]` (deep link).

> 🎓 **Aprende: Expo Go vs Development Build**
> **Expo Go** es una app que instalas de la tienda y ejecuta tu código al instante (ideal para desarrollo, escaneas un QR y listo). Un **Development Build** es tu propia app compilada con librerías nativas personalizadas (se genera en la nube con `eas build`). Para este MVP, Expo Go es suficiente; para publicar en tiendas necesitarás EAS Build. Las push notifications funcionan en ambos.

> ℹ️ **Diferencia clave con la web:** en móvil no hay `localStorage` ni cookies — los tokens van en `expo-secure-store` y se envían manualmente en el header `Authorization: Bearer ...`. El carrito persiste con `AsyncStorage` vía el middleware de persistencia de Zustand.

---

# 13. Manejo de Doble Moneda (USD / VES)

Esta es la feature que hace a DeliLocal realmente venezolano. Los precios se definen en USD pero los clientes pagan frecuentemente en bolívares.

## Reglas

1. **El USD es la moneda base.** Productos, totales y fees se almacenan en USD. El VES es siempre una *conversión de exhibición* (salvo el snapshot `totalVes` del pedido).
2. **Una sola tasa por día** (tabla `ExchangeRate`), obtenida de una API pública del BCV. El admin puede ajustarla manualmente desde `/admin/settings` (la API puede fallar).
3. **Snapshot al comprar:** el pedido guarda la tasa usada. El cliente paga según la tasa *del momento de su pedido*, no la de mañana.
4. **Redondeo:** VES se muestra con 2 decimales usando redondeo half-up. Toda conversión pasa por las utilidades de `@delilocal/shared`.

```typescript
// packages/shared/src/utils/money.ts
// NUNCA uses operaciones de float crudas para mostrar dinero
export function usdToVes(usd: number, rate: number): number {
  // Trabajar en centavos para evitar errores de punto flotante
  return Math.round(usd * 100 * rate) / 100;
}

export function formatUsd(amount: number): string {
  return `$${amount.toFixed(2)}`;
}

export function formatVes(amount: number): string {
  return `Bs. ${amount.toLocaleString('es-VE', {
    minimumFractionDigits: 2, maximumFractionDigits: 2,
  })}`;
}
```

> ⚠️ **Para producción real** considera `Decimal.js` o `dinero.js` para aritmética monetaria exacta en JavaScript. Para el MVP, el truco de centavos de arriba es suficiente, pero debes saber por qué existe el problema.

---

# 14. Features del MVP — Priorización

**Definición de prioridades:**

- **P0 - Crítico**: Sin esto no hay producto. Va en la primera versión.
- **P1 - Alta**: Completa la experiencia core. Segunda iteración del MVP.
- **P2 - Media**: Mejora la usabilidad. Post-MVP cercano.
- **P3 - Baja**: Nice to have. Versiones futuras.

| Feature | Prioridad | Plataforma |
|---|---|---|
| Registro / login (JWT + refresh) | P0 | Web + Móvil |
| Catálogo con categorías e imágenes | P0 | Web + Móvil |
| Carrito persistente | P0 | Web + Móvil |
| Checkout (dirección, método de pago, notas) | P0 | Web + Móvil |
| Creación de pedido con cálculo de totales en servidor | P0 | API |
| Dashboard del negocio con cola de pedidos en tiempo real | P0 | Web |
| Cambio de estados con máquina de estados + WebSocket | P0 | API + Web |
| Seguimiento del pedido en vivo (cliente) | P0 | Web + Móvil |
| Precios en doble moneda USD/VES + tasa del día | P0 | Todas |
| CRUD de productos y categorías (admin) | P1 | Web |
| Reporte y verificación de pago (referencia) | P1 | Todas |
| Push notifications de cambio de estado | P1 | Móvil |
| Direcciones guardadas del cliente | P1 | Todas |
| Historial de pedidos con "repetir pedido" | P1 | Todas |
| Abrir/cerrar tienda (toggle) | P1 | Web |
| Cola de pedidos en app móvil para staff | P2 | Móvil |
| Enviar resumen del pedido por WhatsApp | P2 | Todas |
| Buscador de productos | P2 | Todas |
| Estadísticas básicas del negocio (ventas del día) | P2 | Web |
| Cupones de descuento | P3 | Todas |
| Multi-tienda / marketplace | P3 | Todas |
| Pasarela de pago real | P3 | Todas |
| App para repartidores | P3 | Móvil |

---

# 15. Plan de Desarrollo Paso a Paso

Este es el corazón del documento. Ocho fases ordenadas, cada una con pasos concretos, un **entregable verificable** y una caja *🎓 Aprende* con los conceptos que dominarás. Ritmo estimado: 10-15 horas/semana.

## FASE 1 — Fundamentos y Monorepo (Semanas 1-2)

Sentar las bases: TypeScript, la estructura del monorepo y el esqueleto de las tres apps corriendo.

1. **Setup del monorepo:** `npm create turbo@latest delilocal`, configurar npm workspaces con `apps/*` y `packages/*`.
2. **Crear el paquete shared:** `packages/shared` con los enums (`OrderStatus`, `Role`), tipos base, `ORDER_TRANSITIONS` y utilidades de dinero.
3. **Esqueleto de la API:** `nest new api` dentro de `apps/`, mover a TypeScript estricto, configurar `ConfigModule` con validación de variables de entorno.
4. **Docker Compose para PostgreSQL:** servicio de BD local con volumen persistente (ver sección Comandos).
5. **Prisma inicial:** instalar, pegar el `schema.prisma` de la sección 7, primera migración, `prisma studio` funcionando.
6. **Esqueleto web:** `npx create-next-app@latest web` (App Router, Tailwind, TypeScript), instalar shadcn/ui, página de inicio renderizando.
7. **Esqueleto móvil:** `npx create-expo-app mobile` con Expo Router, correr en Expo Go en tu teléfono.
8. **Scripts raíz:** `npm run dev` en la raíz levanta las tres apps en paralelo vía Turborepo. Commit inicial.

> 🎓 **Aprenderás aquí:** cómo funcionan los workspaces de npm, qué hace Turborepo (cachear builds y orquestar tareas), cómo importar un paquete local (`@delilocal/shared`) desde otra app, y cómo Docker te evita instalar PostgreSQL manualmente. **Fundamento de TypeScript:** interfaces, enums, genéricos básicos — haz los ejercicios del *TypeScript Handbook* en paralelo esta semana.

✅ **Entregable:** `npm run dev` levanta API (responde `GET /api/v1/health`), web (página en localhost:3000) y móvil (Expo Go). Base de datos con tablas creadas. Todo en Git.

## FASE 2 — Autenticación y Usuarios (Semana 3)

Sistema completo de auth: registro, login, tokens, roles y protección de rutas en la API.

1. **AuthModule:** register (bcrypt), login (emite access + refresh), refresh endpoint.
2. **JwtStrategy + JwtAuthGuard:** Passport con `@nestjs/passport`. Proteger un endpoint de prueba.
3. **RolesGuard + decorador `@Roles()`:** crear `CurrentUser` como decorador de parámetro.
4. **UsersModule:** `GET /users/me`, CRUD de direcciones.
5. **Seguridad base:** helmet, CORS, throttler en login, ValidationPipe global.
6. **Seed:** tienda + usuario admin + cliente de prueba + categorías y productos de ejemplo.
7. **Tests unitarios** de AuthService (login válido/inválido, hashing) con Vitest.

> 🎓 **Aprenderás aquí:** cómo funciona JWT por dentro (header.payload.signature, por qué no se puede falsificar), qué es hashing vs cifrado (bcrypt es unidireccional), el patrón Guard de NestJS (≈ middleware de autorización de ASP.NET) y por qué los secretos se validan al arranque.

✅ **Entregable:** desde Swagger/Postman: registrar, loguear, acceder a `/users/me` con token, recibir 401 sin token y 403 con rol incorrecto. Seed cargado.

## FASE 3 — Catálogo y Pedidos, núcleo de la API (Semanas 4-5)

El corazón del negocio: catálogo público y el ciclo completo del pedido con cálculo de totales en servidor.

1. **CatalogModule:** endpoints públicos de lectura (categorías con productos) y CRUD admin con subida de imagen a Cloudinary.
2. **ExchangeRateModule:** servicio que consulta la API del BCV, guarda la tasa diaria, con fallback a tasa manual del admin.
3. **OrdersModule — crear pedido:** el servicio de la sección 8: validación de disponibilidad, totales en servidor, snapshots, número de orden, transacción Prisma.
4. **Consultas de pedidos:** `GET /orders/my` (paginado), `GET /orders/:id` con verificación de propiedad, `GET /orders?status=` para admin.
5. **Cambio de estado:** `PATCH /orders/:id/status` validando contra `ORDER_TRANSITIONS` del paquete shared; cancelación con motivo.
6. **PaymentsModule:** reportar referencia, verificar/rechazar.
7. **Tests:** máquina de estados (todas las transiciones legales e ilegales), cálculo de totales, regla de "precios solo del servidor".

> 🎓 **Aprenderás aquí:** transacciones de base de datos con Prisma (`$transaction`), por qué los snapshots protegen datos históricos, diseño de máquinas de estados, y el principio **never trust the client** aplicado a dinero. Estas son las preguntas de diseño de sistemas que hacen en entrevistas senior.

✅ **Entregable:** flujo completo por Swagger: crear pedido → aparece en cola admin → avanzar por todos los estados → reportar y verificar pago. Tests verdes.

## FASE 4 — WebSockets en Tiempo Real (Semana 6)

El gateway de Socket.IO, autenticación del canal, rooms y emisión de eventos desde los servicios de pedidos.

1. **Constantes de eventos** en `packages/shared/src/constants/ws-events.ts`.
2. **RealtimeGateway** (código de la sección 9): verificación de JWT en handshake, unión automática de admins a `store:admins`.
3. **Mensaje `order:watch`:** verificar propiedad del pedido antes de unir a la room `order:{id}`.
4. **Integrar con OrdersService:** emitir `order:created` al crear y `order:updated` en cada cambio de estado.
5. **Cliente de prueba:** página HTML mínima (o script Node) que se conecta con token y loguea eventos — verificar todo el flujo antes de tocar la web.
6. **Test de integración:** cliente de socket en test que recibe el evento al crear un pedido.

> 🎓 **Aprenderás aquí:** handshake de WebSocket vs HTTP, por qué Socket.IO y no WS crudo (reconexión, rooms, fallbacks), seguridad de canales en tiempo real, y el patrón publicador/suscriptor. Es el equivalente directo de SignalR que conoces de Blazor.

✅ **Entregable:** con dos clientes conectados (uno admin, uno cliente), crear un pedido notifica al admin al instante y cada cambio de estado llega al cliente en vivo.

## FASE 5 — Frontend Web: Cliente (Semanas 7-8)

La experiencia completa del cliente en Next.js: catálogo, carrito, checkout y seguimiento en vivo.

1. **API client tipado** en `lib/api-client.ts` con manejo de tokens y refresh automático.
2. **Auth pages:** login/registro con react-hook-form + zod (schemas compartidos), guardado de tokens, rutas protegidas por middleware.
3. **Catálogo:** Server Component que trae categorías; tarjetas de producto con precio USD/VES; header con tasa del día.
4. **Carrito con Zustand:** añadir/quitar/cantidades, persistencia en localStorage, totales en vivo.
5. **Checkout:** selección de dirección, método de pago, notas, resumen final en doble moneda → crea el pedido y redirige al seguimiento.
6. **Seguimiento en vivo:** página `/orders/[id]` con timeline visual de estados, conectada al WebSocket (join `order:watch`), actualización instantánea.
7. **Historial:** lista de pedidos con badges de estado y botón "repetir pedido".
8. **Pulido UX:** skeletons, estados vacíos, toasts de error, responsive móvil.

> 🎓 **Aprenderás aquí:** React moderno en serio: hooks, Server vs Client Components, data fetching con TanStack Query, estado global mínimo con Zustand, formularios performantes con react-hook-form y el patrón de composición de componentes de shadcn/ui. Esto es el 80% de lo que evalúan en entrevistas de frontend.

✅ **Entregable:** un cliente puede registrarse, pedir, y ver su pedido avanzar en vivo. Todo usable desde el navegador del teléfono.

## FASE 6 — Dashboard del Negocio, Web Admin (Semana 9)

El panel que el dueño del negocio usa todos los días: cola en tiempo real, gestión de menú y configuración.

1. **Layout admin protegido:** verificación de rol, sidebar de navegación.
2. **Dashboard kanban:** columnas por estado (PENDING → READY), tarjetas con items y totales, sonido de nuevo pedido, conexión WebSocket (código de la sección 11).
3. **Acciones de estado:** botones que solo ofrecen transiciones legales según `ORDER_TRANSITIONS`; diálogo de cancelación con motivo.
4. **Detalle de pedido:** items, dirección, notas, referencia de pago con botones verificar/rechazar.
5. **Gestión de menú:** CRUD de productos con subida de imagen (preview), toggle de disponibilidad, orden de categorías.
6. **Configuración:** datos del negocio, delivery fee, tasa manual, toggle abrir/cerrar tienda.

> 🎓 **Aprenderás aquí:** diseño de interfaces de trabajo (no solo de consumo): densidad de información, acciones rápidas, confirmaciones destructivas, y cómo el tiempo real cambia la UX (la pantalla se actualiza sola). También invalidación de caché con TanStack Query tras eventos de socket.

✅ **Entregable:** el negocio gestiona todo su día desde el dashboard: recibir pedidos (con sonido), avanzarlos, verificar pagos y editar el menú. **MVP web completo.**

## FASE 7 — App Móvil con Expo (Semanas 10-12)

Llevar la experiencia del cliente a iOS y Android con push notifications.

1. **Infraestructura móvil:** providers (QueryClient, auth context), API client compartiendo lógica con web, tokens en SecureStore.
2. **Auth screens:** login/registro, persistencia de sesión, redirección según estado de auth en `_layout.tsx`.
3. **Tabs del cliente:** catálogo (lista con imágenes, precios duales), carrito (Zustand + AsyncStorage), pedidos, perfil con direcciones.
4. **Checkout móvil:** mismo flujo que web, reutilizando schemas zod compartidos.
5. **Seguimiento en vivo:** pantalla `order/[id]` con timeline y WebSocket.
6. **Push notifications:** permiso, registro de token, recepción, deep link al pedido al tocarla.
7. **Pantalla staff (P2 opcional):** cola de pedidos con actualización en vivo para rol STAFF.
8. **Build de desarrollo:** `eas build --profile development` y prueba en dispositivo físico.

> 🎓 **Aprenderás aquí:** React Native no es "React en el teléfono": aprenderás sus primitivas (`View`, `Text`, `FlatList`, `Pressable`), cómo funciona la navegación nativa, el ciclo de permisos (notificaciones), y la diferencia entre desarrollo con Expo Go y builds compiladas con EAS. Tu conocimiento de React se transfiere casi completo.

✅ **Entregable:** app instalada en tu teléfono: pedir desde la app, recibir push cuando el negocio actualiza el estado, seguimiento en vivo. **Proyecto completo end-to-end.**

## FASE 8 — Deploy, CI/CD y Portfolio (Semanas 13-14)

Llevar todo a producción real y convertirlo en una pieza de portfolio que consiga entrevistas.

1. **Deploy API + BD:** Railway (o Render): servicio NestJS + PostgreSQL gestionado, variables de entorno, migraciones automáticas en el deploy.
2. **Deploy web:** Vercel conectado al repo (deploy automático por push a `main`), variables de entorno apuntando a la API.
3. **Build móvil:** `eas build` para Android APK instalable (y perfil de producción).
4. **CI con GitHub Actions:** workflow que corre lint + typecheck + tests en cada PR (ver sección CI/CD).
5. **README profesional (en inglés):** qué es, screenshots, diagrama de arquitectura, stack, cómo correrlo, decisiones técnicas.
6. **Video demo (2-3 min, inglés):** graba el flujo completo: cliente pide desde el móvil → suena en el dashboard → estados avanzan en vivo en ambos.
7. **LinkedIn + CV:** publica el proyecto con el video; añade las tecnologías al perfil.

> 🎓 **Aprenderás aquí:** qué significa "producción" de verdad: variables de entorno por ambiente, migraciones en deploy, CORS entre dominios reales, y por qué los pipelines de CI existen (atrapar errores antes de mergear). Además: **cómo presentar un proyecto** — el demo en video vale más que 200 líneas de descripción.

✅ **Entregable:** URLs públicas funcionando, CI verde, README en inglés, video demo. Proyecto listo para mostrar en entrevistas.

## Resumen visual

```text
Sem  1  2  3  4  5  6  7  8  9 10 11 12 13 14
     [──F1──][F2][──F3──][F4][──F5──][F6][───F7───][──F8──]
     Setup  Auth  API   WS  Web   Admin Mobile   Deploy
     mono        núcleo    cliente            push   + CI/CD
```

---

# 16. Mejores Prácticas y Estándares

## Código

- **TypeScript estricto** en las tres apps (`"strict": true`). Prohibido `any` salvo justificación en comentario.
- **Nombres en inglés** en todo el código (variables, funciones, ramas, commits). Comentarios y UI en español.
- **Funciones pequeñas** (< 30 líneas) con un solo propósito. Un service grande se divide por caso de uso.
- **Errores explícitos:** lanzar excepciones tipadas de NestJS (`NotFoundException`, `ForbiddenException`...), nunca retornar `null` silencioso en fallos de negocio.
- **No lógica en controllers:** el controller valida y delega; la lógica vive en services (igual que en tu Clean Architecture).

## Git

- **Conventional Commits:** `feat(orders): add status transition validation`, `fix(api): correct VES rounding`.
- **Ramas por fase/feature:** `feat/orders-module` → PR → `main`. Aunque trabajes solo: practica el flujo profesional.
- **PRs pequeños:** un paso del plan = un PR. Revisa tu propio diff antes de mergear (encontrarás errores).

## Variables de entorno requeridas

| Variable | App | Descripción |
|---|---|---|
| `DATABASE_URL` | api | Conexión PostgreSQL |
| `JWT_SECRET` / `JWT_REFRESH_SECRET` | api | Firma de tokens (32+ chars aleatorios) |
| `CLOUDINARY_*` | api | Cloud name, API key, secret |
| `BCV_API_URL` | api | Fuente de la tasa de cambio |
| `NEXT_PUBLIC_API_URL` | web | URL base de la API |
| `EXPO_PUBLIC_API_URL` | mobile | URL base de la API |

> 🚨 **Jamás** commitees un `.env` real. Solo `.env.example` con valores de muestra. Verifica que `.gitignore` lo incluya desde el primer commit.

---

# 17. Testing

Estrategia pragmática para un proyecto portfolio: cubrir lo que cuesta dinero o rompe el negocio.

| Tipo | Qué cubre | Herramienta | Meta |
|---|---|---|---|
| **Unitarios** | Máquina de estados, cálculo de totales, conversión de moneda, auth service | Vitest | 100% en lógica de dinero y estados |
| **Integración** | Endpoints críticos: crear pedido, cambiar estado, auth | Vitest + supertest (con BD de test en Docker) | Flujo de pedido completo |
| **E2E (opcional)** | Login → pedir → seguir pedido en web | Playwright | 1 flujo feliz |

```typescript
// Ejemplo: test de la máquina de estados
describe('ORDER_TRANSITIONS', () => {
  it('permite el flujo feliz completo', () => {
    expect(canTransition('PENDING', 'CONFIRMED')).toBe(true);
    expect(canTransition('CONFIRMED', 'PREPARING')).toBe(true);
    expect(canTransition('READY', 'DELIVERED')).toBe(true);
  });

  it('rechaza transiciones imposibles', () => {
    expect(canTransition('PENDING', 'DELIVERED')).toBe(false);
    expect(canTransition('CANCELLED', 'CONFIRMED')).toBe(false);
  });
});
```

> 🎓 **Aprende: ¿qué testear cuando el tiempo es limitado?**
> La pregunta correcta no es "¿cuánto coverage?" sino "**¿qué bug me costaría más caro?**". En DeliLocal: totales mal calculados y estados inválidos. Por eso esos tienen 100% de cobertura, mientras que un componente de UI secundario puede quedar sin test. Este criterio (risk-based testing) es lo que diferencia a un ingeniero senior de uno que persigue porcentajes.

---

# 18. CI/CD y Deploy

## Pipeline de CI (GitHub Actions)

```yaml
# .github/workflows/ci.yml
name: CI
on: [pull_request]
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npx turbo run lint typecheck test
```

## Topología de producción

| Pieza | Servicio | Notas |
|---|---|---|
| Web (Next.js) | **Vercel** | Deploy automático desde `main`, gratis |
| API (NestJS) | **Railway** o Render | WebSockets soportados; corre migraciones en release |
| PostgreSQL | **Railway** / Neon / Supabase | Plan gratuito suficiente para el MVP |
| Móvil | **EAS Build** | APK Android gratis; iOS requiere cuenta Apple ($99/año) |
| Imágenes | **Cloudinary** | 25 GB gratis |

> ⚠️ **Atención con Vercel:** sus funciones serverless *no* soportan WebSockets persistentes. Por eso la API (con Socket.IO) vive en Railway/Render, no en Vercel. Esta separación es una decisión de arquitectura deliberada y un excelente tema de conversación en entrevistas.

---

# 19. Comandos de Referencia Rápida

## Setup inicial

```bash
# Crear monorepo
npm create turbo@latest delilocal

# API NestJS
cd apps && nest new api --package-manager npm

# Web Next.js
npx create-next-app@latest web --typescript --tailwind --app
cd web && npx shadcn@latest init

# Móvil Expo
npx create-expo-app@latest mobile --template tabs

# PostgreSQL local con Docker
docker run --name delilocal-db -e POSTGRES_PASSWORD=dev \
  -e POSTGRES_DB=delilocal -p 5432:5432 -d postgres:16
```

## Día a día

| Comando | Qué hace |
|---|---|
| `npm run dev` | Levanta las tres apps en paralelo (Turborepo) |
| `npx prisma migrate dev --name init` | Crea y aplica una migración |
| `npx prisma studio` | Explorador visual de la BD |
| `npx prisma db seed` | Carga datos de prueba |
| `nest g resource orders` | Genera módulo + controller + service |
| `npx turbo run lint typecheck test` | Verificación completa (lo mismo que CI) |
| `npx expo start` | Servidor de desarrollo móvil (QR para Expo Go) |
| `eas build --platform android --profile preview` | APK instalable |

---

# 20. Glosario

| Término | Definición |
|---|---|
| **Monorepo** | Un solo repositorio Git con varios proyectos y paquetes compartidos. |
| **DTO (Data Transfer Object)** | Clase que define la forma exacta de los datos que entran/salen de la API, con validaciones. |
| **Guard** | Componente de NestJS que decide si un request puede continuar (autenticación, roles). |
| **Gateway** | Componente de NestJS que maneja conexiones WebSocket. |
| **Room** | Canal con nombre dentro de Socket.IO para enviar eventos a grupos específicos. |
| **JWT (JSON Web Token)** | Token firmado que contiene la identidad del usuario; el servidor lo verifica sin consultar la BD. |
| **Refresh token** | Token de larga duración usado solo para obtener access tokens nuevos. |
| **Snapshot** | Copia de un dato (precio, tasa, nombre) en el momento de una operación, para preservar el historial. |
| **Máquina de estados** | Modelo que define los estados posibles de una entidad y las transiciones legales entre ellos. |
| **Hidratación** | Proceso por el que React "activa" en el navegador el HTML generado en el servidor. |
| **ORM** | Object-Relational Mapper: traduce entre objetos de código y tablas SQL (Prisma ≈ Entity Framework). |
| **Seed** | Script que carga datos iniciales/de prueba en la base de datos. |
| **Rate limiting** | Límite de peticiones por tiempo para frenar abuso y fuerza bruta. |
| **CI/CD** | Integración Continua (tests automáticos en cada cambio) / Despliegue Continuo (deploy automático). |
| **Push token** | Identificador único del dispositivo que permite enviarle notificaciones push. |
| **Deep link** | Enlace que abre una pantalla específica dentro de la app móvil. |

---

# 21. Recursos y Referencias

## Documentación oficial (tu primera fuente, siempre)

- [NestJS Docs](https://docs.nestjs.com) — excelente, léela por módulos conforme avanzan las fases.
- [Prisma Docs](https://www.prisma.io/docs) — especialmente "Relations" y "Transactions".
- [Next.js Learn](https://nextjs.org/learn) — curso oficial interactivo gratuito, hazlo en la Fase 5.
- [Expo Docs](https://docs.expo.dev) — las guías de Router y Notifications.
- [Socket.IO Docs](https://socket.io/docs/v4) — secciones "Rooms" y "Namespaces".
- [shadcn/ui](https://ui.shadcn.com) — catálogo de componentes para copiar.
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook) — para las semanas 1-2.

## Cuándo consultar qué

| Fase | Recurso principal a estudiar en paralelo |
|---|---|
| F1 | TypeScript Handbook (bases) + docs de Turborepo |
| F2 | NestJS: sección "Authentication" + conceptos JWT |
| F3 | Prisma: relations, transactions + diseño de máquinas de estados |
| F4 | Socket.IO: rooms, namespaces, auth en handshake |
| F5 | Next.js Learn completo + TanStack Query docs |
| F6 | Patrones de dashboards: TanStack Query invalidation |
| F7 | Expo Router + Expo Notifications guides |
| F8 | Docs de Railway, Vercel y EAS Build |

> 💡 **Regla anti-tutorial-hell:** no veas cursos completos "por si acaso". Estudia la documentación *de la fase en la que estás*, construye, y solo busca más cuando te trabes. El proyecto es el curso.

---

**DeliLocal — Guía de Desarrollo v1.0** · Generada como continuación del aprendizaje iniciado con FinTrack
Stack: TypeScript · NestJS · Next.js · Expo · PostgreSQL · Prisma · Socket.IO · Turborepo
Última actualización: octubre 2026
