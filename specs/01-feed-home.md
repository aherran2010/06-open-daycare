# SPEC 01 — Home con el feed de la guardería

> **Estado:** Aprobado
> **Depende de:** —
> **Fecha:** 2026-11-11
> **Objetivo:** Implementar `references/pantallas/feed.dc.html` como la home `/` de la app, replicando su estilo de forma idéntica, sin autenticación ni base de datos.

---

## Por qué existe esta spec

Es la primera pantalla del proyecto y, además, la que fija las bases reutilizables de las otras 14: los tokens de color y tipografía, la estructura del sidebar y el patrón de tarjetas del feed. Si el estilo de esta home no queda idéntico al de referencia, el resto de las pantallas arrastrará el error. Por eso el foco es la fidelidad visual, no la funcionalidad.

---

## Alcance

**In:**

- Reescritura de `app/page.tsx` como home con el layout de `feed.dc.html`.
- Sidebar fijo de escritorio con marca, CTA "Nueva publicación", navegación (Feed activo) y bloque de usuario.
- Encabezado de saludo, compositor y divisor "PUBLICADO HOY".
- Tres tarjetas de post de ejemplo: logro, actividad (con foto) y anuncio.
- Tokens de color y tipografía en `app/globals.css` (`@theme`).
- Carga de Fredoka y Nunito con `next/font/google` en `app/layout.tsx`.
- Datos mock tipados en `app/_data/mock.ts`.
- Comportamiento responsive: sidebar fijo desde `lg` (1024px) y drawer con hamburguesa por debajo.

**Fuera de alcance (para specs futuras):**

- Autenticación, login y sesión real.
- Base de datos o cualquier persistencia.
- Las otras 14 pantallas del catálogo (`ninos`, `avisos`, `crear-publicacion`, `detalle-publicacion`, `familia-feed`, etc.).
- Interactividad real: dar like, comentar, publicar, editar.
- Lógica de fecha dinámica o conteo real de niños.

---

## Modelo de datos

Estructuras nuevas, en `app/_data/mock.ts`. No se conecta a ninguna fuente real.

```ts
export type PostType = "logro" | "actividad" | "anuncio";

export type AvatarKind = "child" | "general";

export type Post = {
  id: string;
  type: PostType;
  authorName: string;        // "Mateo" | "Anuncio general"
  avatarKind: AvatarKind;    // determina inicial vs. ícono
  avatarInitial?: string;    // "M"
  avatarBg: string;          // "#A9D9E8"
  avatarFg: string;          // "#1F7A93"
  time: string;              // "14:20"
  publishedBy: string;       // "publicado por vos"
  recipient: string;         // "Para: familia de Mateo" | "Para: toda la sala"
  body: string;
  photo?: { label: string }; // ej. "Foto · pintando con témperas"
  reactions: number;
  comments: number;
};
```

Constantes de sesión y navegación (mock, estáticas):

```ts
export const session = {
  userName: "Caro Giménez",
  userInitial: "C",
  userRole: "Maestra · Soles",
  greeting: "Buenas, Caro",
  summary: "12 niños · martes 17 jun",
  room: "Sala Soles",
};

export const navItems = [
  { key: "feed", label: "Feed", active: true },
  { key: "ninos", label: "Niños", active: false },
  { key: "avisos", label: "Avisos", active: false },
  { key: "cuenta", label: "Mi cuenta", active: false },
];
```

Tokens de tema en `app/globals.css` (nombres tentativos, valores exactos de la plantilla):

- `--color-canvas: #F6ECDF` (fondo), `--color-surface: #FFFDF9` (tarjetas/sidebar).
- `--color-border: #ECE0D0`, `--color-divider: #E7DAC8`.
- `--color-ink: #3F362E`, `--color-ink-soft: #94887B`, `--color-ink-muted: #A89A8B`.
- `--color-accent: #D9583C`, `--color-accent-strong: #C5503A`, `--color-accent-warm: #E0654A`.
- `--color-accent-soft: #FBE3D8`.
- `--color-staff: #2E89A6`, `--color-family: #7B5FC0`.
- Tipos: logro `#3E9B6C`/`#CFEBD8`, actividad `#2E89A6`/`#C7E7F1`, anuncio `#4E72C8`/`#CCD8F4`.
- Fuentes: `--font-heading` (Fredoka), `--font-body` (Nunito).

---

## Plan de implementación

Cada paso deja el sistema ejecutable.

1. Extender `app/globals.css` con los tokens de color y tipografía en `@theme` y los estilos base (`body` con fondo, color y fuente). Verificación: `npm run dev` levanta sin errores.
2. Actualizar `app/layout.tsx`: cargar Fredoka y Nunito con `next/font/google` exponiendo variables CSS, poner `<html lang="es">` y actualizar el `metadata` (título "OpenDayCare"). Verificación: la home sigue renderizando.
3. Crear `app/_data/mock.ts` con los tipos y los datos de los tres posts, más `session` y `navItems`. Verificación: `npx tsc --noEmit` sin errores.
4. Crear `app/_components/icons.tsx` con los SVG inline usados por la pantalla (sol/logo, plus, home, usuarios, campana, usuario, logout, cámara, corazón, comentario, imagen, megáfono). Verificación: importan y compilan.
5. Crear `app/_components/Sidebar.tsx` como componente cliente: sidebar fijo desde `lg`, barra superior con botón ☰ por debajo, drawer con overlay que cierra al tocar el fondo, al tocar un link y con Escape. Los links son inertes (`href="#"`). Verificación: en escritorio se ve el sidebar; al angostar aparece ☰ y abre el drawer.
6. Crear `app/_components/PostCard.tsx`: cabecera (avatar, nombre, hora, badge de tipo), línea de destinatario, cuerpo, placeholder de foto opcional, barra de reacciones/comentarios y enlace "Editar". Verificación: se renderiza un post de ejemplo.
7. Reescribir `app/page.tsx`: componer `Sidebar` + `<main>` con el encabezado de saludo, el compositor, el divisor "PUBLICADO HOY" y el `map` de posts. Verificación: la home muestra el feed completo.

---

## Criterios de aceptación

- [ ] `npm run dev` levanta sin errores y la home `/` renderiza el feed.
- [ ] `npm run build` completa sin errores.
- [ ] `npx tsc --noEmit` no reporta errores.
- [ ] `npm run lint` no reporta errores.
- [ ] En escritorio (≥1024px) se ve el sidebar fijo de 248px con marca, CTA, navegación y bloque de usuario.
- [ ] Por debajo de 1024px el sidebar se oculta y aparece una barra superior con el botón ☰.
- [ ] El drawer abre con ☰ y cierra al tocar el fondo, al tocar un link y con la tecla Escape.
- [ ] El encabezado muestra "Buenas, Caro" y "12 niños · martes 17 jun".
- [ ] Se renderizan las tres tarjetas: logro, actividad (con placeholder de foto) y anuncio.
- [ ] Los links de navegación, "Nueva publicación", "Editar" y logout no navegan ni cambian de estado.
- [ ] Los botones de reacción y comentario no responden a clics.
- [ ] Tipografías Fredoka (títulos) y Nunito (cuerpo) se aplican correctamente.
- [ ] Una captura de Playwright a 1440px coincide visualmente con `references/screenshots/feed.png`.
- [ ] No hay advertencias de hidratación ni errores en la consola del navegador.

---

## Decisiones tomadas y descartadas

- **Sí:** Tailwind v4 con tokens en `@theme`. Reutilizable por las 14 pantallas restantes.
- **No:** copiar los `style=` inline de la plantilla. Sería píxel-perfecto pero no mantenible.
- **Sí:** `next/font/google` para Fredoka y Nunito. Evita el `<link>` y el layout shift.
- **Sí:** componente `Sidebar` cliente con drawer completo (overlay + cierre por fondo/link/Escape). Es lo que pide el responsive incluido.
- **Sí:** mock tipado en `app/_data/mock.ts`. Fácil de reemplazar por datos reales más adelante.
- **Sí:** links inertes con `href="#"`. Solo se implementa el feed en esta spec.
- **Sí:** textos del encabezado estáticos, copiados de la plantilla. Sin lógica de fecha ni conteo.
- **No:** estado funcional de likes o comentarios. El objetivo es el estilo, no el comportamiento.
- **Sí:** spec en español y estados en español. Consistencia con el proyecto.
- **No:** autenticación y base de datos. No existen todavía en el proyecto.

---

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| Diferencias sutiles de píxel al traducir estilos inline a Tailwind | Comparar contra `references/screenshots/feed.png` en cada paso y ajustar tokens. |
| FOUT o fuentes que no cargan a tiempo | `next/font/google` self-hostea las fuentes y elimina el layout shift. |
| Mismatch de hidratación por el drawer cliente | Controlar la apertura con estado de React y renderizar el overlay solo cuando corresponda. |
| El breakpoint `lg` deja el contenido angosto en tablets | El contenido mantiene `max-width` de 760px para preservar la legibilidad. |

---

## Qué **no** está en esta spec

- Autenticación, login o sesión real.
- Base de datos o cualquier persistencia.
- Las otras 14 pantallas del catálogo.
- Interactividad real (publicar, dar like, comentar, editar).
- Lógica de fecha dinámica o conteo de niños.

Cada uno de esos puntos, si llega, va en su propia spec.
