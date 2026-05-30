# ClickStore — Contexto del proyecto

## Qué es

Sistema de redirección para TikTok. Cuando alguien hace clic en un link de TikTok, el browser nativo de TikTok bloquea redirecciones directas. Este sistema muestra instrucciones para abrir en el navegador real y luego redirige automáticamente a la tienda.

Dominio: **click-here.store** (alojado en Netlify, despliegue manual drag & drop)

## Arquitectura

Un solo archivo `index.html` con router JS del lado del cliente. Sin build step, sin npm, sin Node. El SDK de Supabase se carga vía CDN.

```
clickstore/
├── index.html          ← toda la app (pantallas + admin + lógica)
└── netlify.toml        ← solo catch-all redirect a index.html
```

## Stack

- **Frontend:** HTML + CSS + JS vanilla (sin framework)
- **Base de datos:** Supabase (PostgreSQL con RLS)
- **Auth:** Supabase Auth (email + password)
- **Hosting:** Netlify (drag & drop manual)
- **Fonts:** Bebas Neue + DM Sans (Google Fonts CDN)
- **SDK:** `@supabase/supabase-js@2` vía CDN (UMD minificado)

## Supabase

**Proyecto:** `rxrrubijeoybauuwfgpr`
**URL:** `https://rxrrubijeoybauuwfgpr.supabase.co`

### Tabla `stores`

```sql
create table stores (
  key text primary key,          -- slug de la URL (ej: "flapicat")
  name text not null,            -- nombre visible
  accent text not null,          -- color hex (ej: "#FF6B2B")
  redirect_url text not null,    -- URL destino
  icon_emoji text,               -- emoji del ícono
  lang text default 'es'         -- idioma: "es" o "en"
);
```

### RLS policies

```sql
-- Lectura pública (necesaria para que funcionen los redirects)
create policy "lectura publica" on stores for select using (true);

-- Escritura solo para admin autenticado
create policy "escritura admin" on stores for all using (auth.role() = 'authenticated');
```

### Admin user

- **Email:** `samu@admin.local`
- **Password:** `203820`
- El campo "Usuario" en el login acepta `samu` → internamente se mapea a `samu@admin.local` via `USERS = { samu: 'samu@admin.local' }`

## Tiendas actuales (defaults en Supabase)

| key | name | accent | redirect_url | emoji | lang |
|-----|------|--------|-------------|-------|------|
| flapicat | FlapiCat | #FF6B2B | https://flapicat.com | 🐱 | es |
| daisybags | Daisy's Bags | #C4874A | https://daisybags.store | 👜 | es |
| chloe | Chloe Handbags | #9B5DE5 | https://chloehandbags.com | 🎸 | es |
| cozyrest | CozyRest | #6BBF8E | https://cozyrest.com | 🛋️ | es |

## Cómo funciona el router

1. Lee la ruta: soporta `/flapicat` (pathname) y `?s=flapicat` (query param)
2. Si la ruta es `admin` → flujo de admin
3. Si no hay ruta → 404
4. Consulta Supabase: `sb.from('stores').select('*').eq('key', route).single()`
5. Detecta si está en browser TikTok con user agent (`/TikTok|musical_ly|BytedanceWebview/`)
   - **TikTok:** muestra pantalla de instrucciones (3 pasos)
   - **Normal:** muestra spinner y redirige tras 1.2s

## Admin dashboard

**Acceso:** `click-here.store/admin`
**Credenciales:** usuario `samu`, contraseña `203820`

### Seguridad

- El HTML del dashboard **no existe en el DOM** hasta después del login exitoso
- Se construye dinámicamente con `buildAdminDOM()` solo tras auth
- Al logout, todos los elementos admin se eliminan del DOM con `.remove()`
- Sesión guardada en `sessionStorage` (se borra al cerrar la pestaña)
- Auth real server-side via Supabase Auth (no hashes en cliente)

### Funcionalidades del dashboard

- Ver todas las tiendas en grid de tarjetas
- **Añadir** tienda: slug, nombre, color (picker + hex), URL, emoji, idioma
- **Editar** tienda: mismos campos, slug readonly
- **Eliminar** tienda: con confirmación
- **Preview** tienda: modal con frame de teléfono, tabs para ver pantalla TikTok y pantalla de redirección

## Pantallas en el HTML estático

Solo estas 4 existen en el DOM inicial (el inspector solo ve esto):

1. `#warning` — pantalla TikTok (instrucciones 3 pasos)
2. `#redirecting` — spinner de redirección
3. `#not-found` — 404
4. `#admin-login` — formulario de login

El dashboard y los modales se inyectan dinámicamente tras auth.

## Idiomas

Cada tienda tiene su propio `lang` (`es` o `en`). Los textos de las pantallas (pasos del warning, label de redirección, footer) cambian según el idioma de la tienda. El objeto `T` en el JS tiene ambas traducciones.

## netlify.toml

```toml
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

Solo un catch-all. No hay funciones Netlify (se intentó pero daban 502 por problemas con el bundler en despliegues manuales). Todo va por Supabase directamente desde el cliente.

## Historial de decisiones importantes

- **Se intentó Netlify Functions + Netlify Blobs** para el storage → 502 errors en despliegues manuales, se descartó
- **Se intentó hash SHA-256 client-side** para auth → visible en source, se reemplazó por Supabase Auth
- **El anon key de Supabase está en el cliente** → esto es correcto y seguro por diseño; la seguridad la controla RLS en el servidor
- **SDK vía CDN** en lugar de npm → evita dependencias de build, compatible con drag & drop en Netlify
- **DOM del admin se construye dinámicamente** → nadie puede ver el HTML del dashboard sin autenticarse
