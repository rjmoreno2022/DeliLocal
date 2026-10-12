# AGENTS.md — DeliLocal

> Instrucciones obligatorias para cualquier agente de IA que trabaje en este repositorio.
> Lee este archivo COMPLETO antes de generar código. La guía de diseño completa está en `docs/DELILOCAL-GUIDE.md`.

## 1. Qué es este proyecto

**DeliLocal**: plataforma de pedidos en línea **mono-tienda** (un negocio local por despliegue) con seguimiento en tiempo real. Cliente (web + móvil) hace pedidos; el negocio los gestiona desde un dashboard web en vivo. Contexto Venezuela: precios en USD con conversión a VES según tasa del día; pagos por confirmación manual (pago móvil, Zelle, efectivo) — NO hay pasarela de pago.

## 2. Stack — NO TE DESVÍES

| Capa | Tecnología |
|---|---|
| Lenguaje | TypeScript 5 estricto (`"strict": true`) en TODO |
| Monorepo | Turborepo + npm workspaces |
| API | NestJS 10 + Prisma 5 + PostgreSQL 16 |
| Web | Next.js 14 (App Router) + Tailwind CSS + shadcn/ui |
| Móvil | Expo SDK 51 (React Native) + Expo Router |
| Tiempo real | Socket.IO vía NestJS Gateways (namespace `/realtime`) |
| Auth | JWT access (15 min) + refresh (30 días), bcrypt cost 12 |
| Validación | Zod (shared) + class-validator (DTOs API) |
| Estado cliente | Zustand (carrito) + TanStack Query (datos) |
| Formularios | react-hook-form + zod |
| Imágenes | Cloudinary |
| Testing | Vitest |
| Deploy | Vercel (web) + Railway (API + BD) + EAS Build (móvil) |

**PROHIBIDO:** proponer librerías alternativas (Redux, TypeORM, Express puro, Firebase, styled-components, etc.). Si crees que algo del stack no sirve, explica el problema y espera aprobación del usuario antes de cambiarlo.

## 3. Estructura del monorepo — RESPÉTALA

```text
delilocal/
├── apps/
│   ├── api/        # NestJS. Módulos por dominio: auth, users, catalog,
│   │               # orders, payments, notifications, exchange-rate, realtime, prisma
│   ├── web/        # Next.js App Router: app/(shop)/, app/(admin)/admin/, app/(auth)/, components/, lib/
│   └── mobile/     # Expo Router: app/(tabs)/, app/order/[id].tsx, app/(auth)/, app/(admin)/
├── packages/
│   └── shared/     # @delilocal/shared — types/, schemas/ (zod), constants/, utils/
└── turbo.json
```

**Reglas de dependencias (inviolables):**
- `apps/*` PUEDEN importar de `@delilocal/shared`.
- `packages/*` NUNCA importan de `apps/*`.
- Las apps NUNCA importan entre sí.

**Lo que vive SIEMPRE en `shared` (fuente única de verdad):**
- Enum `OrderStatus` y tabla `ORDER_TRANSITIONS` (máquina de estados).
- Constantes de eventos WebSocket (`ws-events.ts`): `order:created`, `order:updated`, `payment:reported`, `store:toggled`.
- Schemas Zod usados en formularios Y validación de API.
- Utilidades de dinero (`usdToVes`, `formatUsd`, `formatVes`).

## 4. Reglas de negocio innegociables

1. **Los precios JAMÁS se toman del request del cliente.** El cliente envía `productId` + `quantity`; el servidor busca el precio real en la BD y calcula los totales.
2. **Snapshots:** `OrderItem` copia nombre y precio del producto al comprar. `Order` guarda `exchangeRate` y `totalVes` calculados al momento de la compra. Los históricos NUNCA cambian.
3. **Máquina de estados del pedido:** `PENDING → CONFIRMED → PREPARING → READY → DELIVERED` (o `CANCELLED` con motivo). Toda transición se valida contra `ORDER_TRANSITIONS` en el servidor. Estados finales no tienen salida.
4. **Disponibilidad:** productos con `isAvailable = false` no entran en pedidos nuevos (validado en servidor).
5. **Propiedad de recursos:** un CUSTOMER solo ve sus pedidos. Verificar `order.customerId === user.sub` en el service, no solo en el guard.
6. **Doble moneda:** USD es la moneda base de almacenamiento; VES es conversión de exhibición con la tasa del día (tabla `ExchangeRate`, fallback manual del admin).
7. **Todo cambio de estado emite:** evento WebSocket a `order:{id}` y `store:admins` + push notification vía `expo-server-sdk`.

## 5. Estándares de código

- Nombres en **inglés** (variables, funciones, ramas, commits). Comentarios y textos de UI en **español**.
- Sin `any` salvo justificación en comentario.
- **Sin lógica en controllers:** validan y delegan al service.
- Errores con excepciones tipadas de NestJS (`NotFoundException`, `ForbiddenException`, `BadRequestException`). Nunca `null` silencioso en fallos de negocio.
- Commits con Conventional Commits: `feat(orders): ...`, `fix(api): ...`.
- Variables de entorno validadas al arranque (fail fast). NUNCA commitear `.env` real.
- Dinero: `Decimal` en Prisma, conversiones solo vía utils de `shared`. Nunca floats crudos para mostrar dinero.
- Queries calientes indexadas: `(storeId, status)` en Order, `(customerId, createdAt)` en Order.

## 6. Seguridad (checklist obligatorio)

- [ ] `ValidationPipe` global con `whitelist: true` + `forbidNonWhitelisted: true`
- [ ] Helmet + CORS restringido a orígenes conocidos
- [ ] Rate limiting: 10 req/min en `/auth/login` (`@nestjs/throttler`)
- [ ] Rol siempre del JWT firmado, nunca del body
- [ ] Handshake WebSocket verifica JWT; `order:watch` verifica propiedad del pedido
- [ ] bcrypt cost 12; passwordHash nunca en respuestas ni logs
- [ ] Prohibido `$queryRawUnsafe` con interpolación de strings

## 7. Web (Next.js) — convenciones

- Server Components por defecto; `'use client'` lo más abajo posible en el árbol.
- Data fetching: `fetch` en Server Components; TanStack Query en cliente.
- Precios SIEMPRE en doble moneda: `$5.00 · Bs. 542,50`.
- Skeletons de carga, estados vacíos diseñados, errores en español humano.
- Dashboard admin: columnas kanban por estado, sonido al llegar pedido nuevo, botones solo con transiciones legales.
- Responsive mobile-first.

## 8. Móvil (Expo) — convenciones

- Tokens en `expo-secure-store`, enviados como `Authorization: Bearer`.
- Carrito persiste con AsyncStorage (middleware de Zustand).
- Push: registrar `ExpoPushToken` en `POST /notifications/push-token` tras login; deep link a `/order/[id]` al tocar notificación.
- Desarrollo con Expo Go; builds con EAS.

## 9. Testing — qué es obligatorio testear

- 100% de cobertura en: máquina de estados, cálculo de totales, conversión de moneda.
- Integración: crear pedido (regla de precios del servidor), cambio de estado, auth.
- Todo paso del plan que diga "tests" los incluye; no los omitas.

## 10. Flujo de trabajo con el usuario

1. El usuario trabaja **una fase a la vez** según el plan de `docs/DELILOCAL-GUIDE.md` (8 fases). No generes código de fases futuras salvo que se pida.
2. Explica cada decisión no obvia con comentarios en el código — el usuario está aprendiendo.
3. No agregues dependencias sin justificarlas y pedir confirmación.
4. Al terminar cada tarea, indica los **comandos para verificar** que funciona.
5. Ante duda entre "rápido" y "correcto según este documento": correcto.

## 11. Variables de entorno

| Variable | App | Descripción |
|---|---|---|
| `DATABASE_URL` | api | Conexión PostgreSQL |
| `JWT_SECRET` / `JWT_REFRESH_SECRET` | api | Firma de tokens (32+ chars) |
| `CLOUDINARY_CLOUD_NAME` / `CLOUDINARY_API_KEY` / `CLOUDINARY_API_SECRET` | api | Imágenes |
| `BCV_API_URL` | api | Fuente de tasa USD/VES |
| `NEXT_PUBLIC_API_URL` | web | URL base de la API |
| `EXPO_PUBLIC_API_URL` | mobile | URL base de la API |

## 12. Comandos frecuentes

```bash
npm run dev                              # levanta las 3 apps (Turborepo)
npx prisma migrate dev --name <nombre>   # migración
npx prisma db seed                       # datos de prueba
npx prisma studio                        # explorador de BD
npx turbo run lint typecheck test        # verificación completa (= CI)
npx expo start                           # dev server móvil
docker run --name delilocal-db -e POSTGRES_PASSWORD=dev -e POSTGRES_DB=delilocal -p 5432:5432 -d postgres:16
```

---

*Referencia completa de diseño, esquema Prisma, endpoints, código de ejemplo y plan de 8 fases: `docs/DELILOCAL-GUIDE.md` (v1.0).*
