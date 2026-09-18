# Luxz Store

Tienda en línea de una sola página (HTML + JavaScript, sin frameworks) conectada a [Supabase](https://supabase.com) como backend. Pensada para publicarse gratis en **GitHub Pages**.

---

## ✨ Funcionalidades

### Para clientes
- Registro e inicio de sesión (Supabase Auth).
- Catálogo con categorías, badges de "Verificado", "Más vendido", "Nuevo" y "En oferta".
- Ficha de producto con varios planes/presentaciones y precios (ej: 1kg, 2kg, 3kg).
- Carrito de compra en 3 pasos: opción y cantidad → datos del comprador → método de pago.
- Billetera interna (saldo) con recarga vía métodos de pago manuales (ej. Binance Pay), verificados por el dueño de la tienda con el ID de la transacción.
- Historial de saldo y de pedidos, con el código/licencia asignado a cada compra (cuando aplica) y botón para copiarlo.
- Botón flotante de soporte que abre WhatsApp con un mensaje pre-cargado.
- Enlaces a redes sociales (WhatsApp, Discord, TikTok, Instagram) en el pie de página.

### Para Administradores
- Panel con: Resumen (estadísticas de ventas), Productos, Categorías, Usuarios, Pedidos, Accesos y Licencias.
- Crear/editar productos: nombre, categoría, descripción, imagen, planes con precio y stock, etiqueta destacada.
- Gestión de categorías (agregar/quitar).
- Suspender usuarios o ascenderlos a administrador.
- Cambiar el estado de cada pedido (Pendiente / Enviado / Entregado / Cancelado).
- Ver el historial de inicios de sesión.
- Cargar y administrar licencias/códigos por producto y plan (uno por línea).

### Exclusivo del Owner (dueño total)
- Todo lo anterior, más:
  - Eliminar productos y categorías.
  - Ascender/degradar usuarios a Cliente, Admin u Owner.
  - Ajustar manualmente el saldo de cualquier usuario (con motivo).
  - Confirmar o rechazar solicitudes de recarga de saldo.
  - Administrar los métodos de pago que ven los clientes (nombre + instrucciones + visible/oculto).

### Sistema de licencias
- Cada producto/plan puede tener un inventario de códigos únicos.
- Si tiene códigos cargados, el **stock se calcula solo** a partir de cuántos quedan disponibles (ya no hace falta escribirlo a mano).
- Al comprar, se asigna un código automáticamente y de forma atómica (no se puede vender el mismo código dos veces).
- Si un producto no tiene licencias cargadas, sigue funcionando con el campo de stock manual de siempre.
- El código solo se revela al cliente cuando el pago ya está confirmado (pagos con saldo se ven al instante; pagos manuales, cuando el Owner/Admin cambia el estado del pedido).

---

## 🧱 Tech stack

- **Frontend:** HTML + CSS + JavaScript puro (sin build step, un solo archivo `index.html`).
- **Backend:** [Supabase](https://supabase.com) — Auth, Postgres, Row Level Security (RLS) y funciones `SECURITY DEFINER` para las operaciones sensibles (compras, ajustes de saldo, licencias).
- **Hosting:** GitHub Pages (estático).

No hay servidor propio: toda la lógica de negocio sensible vive en funciones de Postgres dentro de Supabase, nunca en el navegador.

---

## 🚀 Instalación desde cero

### 1. Crea el proyecto en Supabase
Ve a [supabase.com](https://supabase.com) → **New project** (plan gratuito).

### 2. Corre las migraciones SQL, en este orden
En **Supabase → SQL Editor**, pega y ejecuta cada archivo completo, uno a la vez:

| # | Archivo | Qué hace |
|---|---------|----------|
| 1 | `supabase-setup.sql` | Tablas base (perfiles, productos, pedidos, accesos), seguridad (RLS) y la función de compra. |
| 2 | `fix-registro.sql` | Crea el perfil automáticamente al registrarse (evita errores de permisos). |
| 3 | `migracion-destacado-categorias.sql` | Etiquetas destacadas y categorías administrables. |
| 4 | `nivelacion-completa.sql` | Rol Owner, sistema de saldo con historial, métodos de pago, recargas y licencias. |
| 5 | `migracion-stock-licencias.sql` | Conecta el stock al inventario de licencias. |

> Los demás archivos `.sql` del repositorio (`migracion-owner-saldo-pagos.sql`, `migracion-pedidos-manuales.sql`, `migracion-binance-recargas.sql`, `migracion-licencias.sql`, `fix-trigger-rol.sql`, `diagnostico.sql`, etc.) son versiones intermedias o utilidades de diagnóstico que ya quedaron incluidas en `nivelacion-completa.sql`. Se conservan como historial, pero **no hace falta correrlas** en una instalación nueva.

### 3. Conecta tus claves
En Supabase: **Project Settings → API**. Copia:
- `Project URL`
- `anon public key`

Ábrelas en `index.html` y reemplaza:
```js
const SUPABASE_URL = 'https://TU-PROYECTO.supabase.co';
const SUPABASE_ANON_KEY = 'TU-ANON-KEY';
```

### 4. (Opcional) Configura WhatsApp y redes sociales
En el mismo bloque de configuración de `index.html`:
```js
const WHATSAPP_NUMBER = '584121234567'; // solo dígitos, con código de país
const SOCIAL_LINKS = {
  whatsapp:  'https://chat.whatsapp.com/tu-grupo',
  discord:   'https://discord.gg/tu-servidor',
  tiktok:    'https://www.tiktok.com/@tu-usuario',
  instagram: 'https://www.instagram.com/tu-usuario',
};
```

### 5. Desactiva la confirmación por correo (recomendado)
En **Supabase → Authentication → Providers → Email**, apaga **"Confirm email"** para que los usuarios queden con sesión iniciada apenas se registran.

### 6. Publica en GitHub Pages
Sube `index.html` a la raíz de tu repositorio y activa GitHub Pages en **Settings → Pages**.

### 7. Vuélvete Owner
Regístrate normal desde la web con tu cuenta, y luego en **Supabase → SQL Editor**:
```sql
update public.profiles set role = 'owner' where email = 'tu@correo.com';
```

---

## 👤 Roles

| Rol | Puede... |
|---|---|
| **Cliente** | Comprar, recargar saldo, ver su historial y perfil. |
| **Admin** | Todo lo del Cliente, más: gestionar productos, categorías, usuarios (suspender/ascender a admin), pedidos, licencias y ver accesos. |
| **Owner** | Todo lo del Admin, más: eliminar productos/categorías, gestionar cualquier rol (incluido crear otros Owners), ajustar saldos manualmente, confirmar/rechazar recargas y administrar los métodos de pago. |

---

## ⚠️ Notas y limitaciones

- **El saldo no usa una pasarela de pago automática.** Las recargas se verifican manualmente: el cliente paga por fuera (ej. Binance Pay) y pega el ID de su transacción; el Owner lo revisa y confirma con un clic. Para automatización 100% real (webhooks firmados) haría falta una cuenta Binance Pay Merchant y un pequeño backend (Supabase Edge Functions) que nunca expone claves secretas en el frontend.
- La `anon key` de Supabase es segura de publicar en el frontend — no es secreta, y todos los permisos reales están controlados por Row Level Security en la base de datos.
- Cancelar un pedido de pago manual no repone automáticamente el stock/licencia reservada; hay que ajustarlo manualmente si ocurre.

---

## 📄 Licencia

Uso libre para este proyecto personal.
