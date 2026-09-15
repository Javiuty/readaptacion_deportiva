# Nerea Valdés — Readaptación Deportiva

Landing de una entrenadora personal especializada en readaptación deportiva de lesiones.
Sitio estático de una sola página, tema oscuro y sin librerías de cliente.

> **Datos de ejemplo.** El nombre, el teléfono y el Instagram son inventados: cámbialos
> en `src/consts.ts` y `src/components/Contact.astro` antes de publicar.

## Stack

| Pieza | Versión | Notas |
| :---- | :------ | :---- |
| [Astro](https://astro.build) | 7.0.9 | Salida `static`, una sola ruta |
| [Tailwind CSS](https://tailwindcss.com) | 4.x | Vía `@tailwindcss/vite`, sin `tailwind.config.js` |
| TypeScript | — | Preset `astro/tsconfigs/strict` |
| Fuente Archivo | — | Alojada en el propio proyecto (`public/fonts/`) |
| sharp | 0.35.x | Motor de `astro:assets`; sin él el build falla |

No hay framework de UI (React, Vue…): los componentes son `.astro` y la interactividad
es TypeScript sin librerías.

## Requisitos

- Node.js **>= 22.12.0**
- pnpm (el repo trae `pnpm-lock.yaml`)

## Puesta en marcha

```sh
pnpm install
pnpm dev        # http://localhost:4321
```

## Comandos

| Comando | Qué hace |
| :------ | :------- |
| `pnpm install` | Instala dependencias |
| `pnpm dev` | Servidor de desarrollo en `localhost:4321` |
| `pnpm build` | Compila el sitio a `./dist/` |
| `pnpm preview` | Sirve `./dist/` para revisar el build |
| `pnpm astro ...` | CLI de Astro (`astro add`, `astro check`…) |

`astro check` no está preinstalado: la primera vez te ofrecerá añadir `@astrojs/check`.

## Estructura

```text
├── design/                  Diseño de referencia (no entra en el build)
├── public/
│   ├── fonts/               Archivo en woff2 (normal + itálica, 3 subsets)
│   └── favicon.*            Icono de pesa rusa
├── src/
│   ├── assets/              Fotos originales, optimizadas en el build
│   ├── components/          Una sección de la página por componente
│   ├── layouts/Layout.astro <head>, metadatos, precarga de fuentes
│   ├── pages/index.astro    Única ruta + todo el JS de la página
│   ├── styles/global.css    Tokens de diseño, utilidades y componentes
│   └── consts.ts            Datos de contacto y enlaces del menú
└── astro.config.mjs
```

`index.astro` monta las secciones en orden: Hero → Marquee → Services → Method → About →
Testimonials → Pricing → Faq → Contact, con Nav, Footer y el botón flotante de WhatsApp
alrededor.

## Sistema de diseño

Todo el color vive en el bloque `@theme` de `src/styles/global.css`. Cambia un token ahí y
se propaga a todo el sitio; no hay colores sueltos en los componentes salvo sombras y
tramas puntuales.

### Paleta (tema oscuro)

| Token | Valor | Uso |
| :---- | :---- | :-- |
| `--color-bg` | `oklch(0.15 0.018 325)` | Fondo base |
| `--color-bg2` | `oklch(0.185 0.022 325)` | Bandas alternas de sección |
| `--color-surface` | `oklch(0.215 0.025 325)` | Tarjetas |
| `--color-surface2` | `oklch(0.25 0.028 325)` | Tarjeta destacada (plan recomendado) |
| `--color-line` | `oklch(0.32 0.03 325 / 0.55)` | Bordes y separadores |
| `--color-text` | `oklch(0.96 0.008 325)` | Texto principal |
| `--color-muted` | `oklch(0.74 0.025 325)` | Texto secundario |
| `--color-accent` | `oklch(0.74 0.115 330)` | Malva de marca |
| `--color-on-accent` | `oklch(0.17 0.03 330)` | Texto sobre malva |
| `--color-accent2` | `oklch(0.74 0.055 262)` | Azul de marca (contornos, FAQ) |
| `--color-wa` | `oklch(0.72 0.17 155)` | Verde de WhatsApp |

Los acentos son las versiones aclaradas del malva `#876987` y el azul `#525e76`
corporativos, que sobre fondo oscuro no dan contraste suficiente. Todos los pares de
texto sobre fondo superan **WCAG AAA** (7:1); el mínimo de la paleta es 6.6:1 en el rojo
de error del formulario.

### Utilidades propias

| Clase | Para qué |
| :---- | :------- |
| `display` | Titulares: Archivo 900, expandida al 125 %, en mayúsculas |
| `text-outline` | Texto en contorno (marquesina) |
| `container-x` | Contenedor centrado de 1140 px con margen lateral |
| `btn` + `btn-accent` / `btn-wa` / `btn-ghost` | Botones, con brillo al pasar por encima |
| `ph` | Marcador de posición de imagen (trama diagonal + etiqueta) |
| `marker-list` / `check-list` | Listas con viñeta rombo / con check |
| `quote-text` | Cita con comilla decorativa |

### Puntos de ruptura

Además de los de Tailwind, hay dos a medida: `xs` (560 px) y `nav` (980 px), este último
es donde el menú pasa a hamburguesa.

## Imágenes

Las fotos viven en `src/assets/` (no en `public/`) y se montan con `<Image>` de
`astro:assets`. En el build, Astro genera WebP en dos anchos (480 y 928 px) y escribe el
`srcset`; el `sizes` de cada componente le dice al navegador cuál bajar. Las dos juntas
pasan de 1,4 MB a 136 KB en escritorio y 57 KB en móvil.

La del hero va con `loading="eager"` y `fetchpriority="high"` porque es el LCP; la de
"Sobre mí" se queda con el `lazy` que pone Astro por defecto.

Para cambiar una foto, sustituye el fichero en `src/assets/` y ajusta el `alt`. Si la
nueva no es 4:5, cambia también el `aspect-[4/5]` de la clase o se recortará.

## Interactividad

Todo el JavaScript está en el `<script>` de `src/pages/index.astro`:

- **Nav** — borde al hacer scroll, enlace activo según la sección visible y barra de
  progreso de lectura (variable CSS `--scroll-progress`).
- **Menú móvil** — apertura y cierre de la hamburguesa.
- **Aparición al hacer scroll** — un `IntersectionObserver` añade `.in` a los elementos
  con clase `.reveal`. El estado oculto solo se activa cuando el observer demuestra que
  funciona (clase `io-ok` en `<html>`), así que si falla el contenido se ve igualmente.
  `.reveal-group` escalona los hijos.
- **FAQ** — acordeón de apertura única.
- **Formulario** — validación en cliente y estado "enviado".

Las animaciones respetan `prefers-reduced-motion`.

## Editar el contenido

| Qué quieres cambiar | Dónde |
| :------------------ | :---- |
| Teléfono, WhatsApp, enlaces del menú | `src/consts.ts` |
| Nombre de marca y monograma | `src/components/Brand.astro` |
| Título y descripción para buscadores | `src/layouts/Layout.astro` |
| Textos de una sección | El componente con ese nombre en `src/components/` |
| Servicios, pasos del método, precios, FAQ, testimonios | Array `const` al principio de cada componente |
| Instagram | `src/components/Contact.astro` |
| Fotos | Ficheros en `src/assets/`, montadas con `<Image>` en `Hero.astro` y `About.astro` |
| Colores | Bloque `@theme` en `src/styles/global.css` |

