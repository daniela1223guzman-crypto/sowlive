# Live Commerce TikTok

Reserva automática de stock limitado durante transmisiones en vivo de TikTok: comentas la palabra clave del producto en el chat del LIVE, y el sistema reserva la unidad al primer usuario que la escribió — en tiempo real.

Diseñado para venderse desde el celular como siempre: sin OBS, sin software de transmisión. Solo abres el Panel en otra pantalla mientras transmites con la app de TikTok.

Ver `ARQUITECTURA.md` para el diseño completo del sistema (stack, módulos, esquema de datos y manejo de concurrencia).

## Estructura

```
live-commerce/
├── ARQUITECTURA.md        # documento de diseño completo
├── backend/                 # API + motor "Sniper" (Node.js/TypeScript)
│   ├── schema.sql            # DDL de PostgreSQL + función de reserva atómica
│   └── src/
│       ├── server.ts         # arranque de Fastify + Socket.IO
│       ├── db.ts             # pool de PostgreSQL
│       ├── tiktokSniper.ts   # conexión a TikTok LIVE y matching de comentarios
│       └── routes/           # productos, sesiones, reservas (CSV)
└── frontend/                 # Panel del vendedor (React/Vite)
    └── src/
        ├── pages/Panel.tsx    # catálogo, control del LIVE, feed de ganadores
        └── lib/               # cliente API y cliente WebSocket
```

## Puesta en marcha

### 1. Base de datos

```bash
createdb live_commerce
psql live_commerce -f backend/schema.sql
```

Crea manualmente tu primer vendedor (todavía no hay pantalla de registro):

```sql
INSERT INTO vendedores (nombre, email, tiktok_username)
VALUES ('Mi Tienda', 'yo@ejemplo.com', 'mi_usuario_tiktok')
RETURNING id;
```

Guarda el `id` que devuelve — lo necesitas en el paso 3.

### 2. Backend

```bash
cd backend
cp .env.example .env    # completa DATABASE_URL con tus credenciales
npm install
npm run dev              # http://localhost:3000
```

### 3. Frontend

```bash
cd frontend
cp .env.example .env    # pega el vendedorId del paso 1 en VITE_VENDEDOR_ID
npm install
npm run dev               # http://localhost:5173
```

### 4. Uso

1. Abre `http://localhost:5173`, agrega tus productos con su palabra clave (ej. `1`, `rojo-m`)
2. Transmite en vivo normalmente desde la app de TikTok en tu celular
3. En el Panel (otra pantalla: tablet, laptop), escribe tu usuario de TikTok (sin @) y pulsa **Iniciar LIVE**
4. Los comentarios que coincidan exactamente con una palabra clave activa reservan la unidad al primer usuario, y aparecen al instante en el Panel
5. Al terminar, exporta el CSV de ganadores desde el panel

## Pendiente para producción

- Autenticación real de vendedores (hoy el `vendedorId` se pega a mano vía variable de entorno) — próximo paso natural si se convierte en un producto multi-vendedor con planes y prueba gratuita
- Reconexión automática del Sniper si TikTok corta la conexión WebCast a media transmisión
- Rate-limiting / anti-spam en el matching de comentarios si el volumen de chat es muy alto
- Despliegue: el backend necesita un host que soporte procesos corriendo de forma continua (Railway, Render, Fly.io) — no funciona bien en plataformas serverless como Vercel, porque debe mantener la conexión abierta con TikTok mientras dura el LIVE. El frontend sí puede ir en Vercel/Netlify. Supabase es una buena opción para la base de datos y para el login cuando se agregue autenticación.

