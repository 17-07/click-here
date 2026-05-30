# 🚀 ClickStore — TikTok Redirect System

## ¿Cómo funciona?

1. Usuario ve tu video en TikTok y clica el link (ej: `tudominio.com/flapicat`)
2. **Si está en TikTok** → ve las instrucciones de "abre en navegador"
3. **Si está en navegador** → redirect automático a tu tienda en 1.2s

---

## Setup en Netlify (gratis, 5 minutos)

1. Ve a [netlify.com](https://netlify.com) y crea cuenta gratis
2. Click en "Add new site" → "Deploy manually"
3. **Arrastra la carpeta entera** `clickstore/` al área de drop
4. Listo — te da un dominio `.netlify.app`

Opcional: conecta tu propio dominio en Site settings → Domain management

---

## Añadir nuevas tiendas

Abre `index.html` y busca el objeto `STORES`. Añade:

```js
mitienda: {
  name: 'Mi Tienda',
  accent: '#FF5500',           // color principal
  accentA30: 'rgba(255,85,0,0.3)',
  accentA50: 'rgba(255,85,0,0.5)',
  redirectUrl: 'https://mitienda.com',
  iconEmoji: '🔥',
},
```

Luego el link de TikTok sería: `tudominio.netlify.app/mitienda`

---

## Links de ejemplo

| TikTok link | Redirige a |
|---|---|
| `tudominio.com/flapicat` | theflapicat.com |
| `tudominio.com/daisybags` | daisybags.store |
| `tudominio.com/chloe` | chloehandbags.com |
| `tudominio.com/cozyrest` | cozyrest.com |

---

## Para cambiar el redirect delay

Busca en el JS:
```js
setTimeout(() => {
  window.location.href = store.redirectUrl;
}, 1200); // cambia este número (milisegundos)
```
