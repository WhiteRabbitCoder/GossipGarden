---
name: presentation-storybook-design
description: Crayon Storybook Design System adapted for Gossip Garden Presentation (Next.js/React). Enforces the paper-like, playful, and warm aesthetic for all UI elements. Trigger whenever creating, styling, or fixing React components, pages, or animations in the gossip-garden-presentation app.
---

# Gossip Garden Presentation — Storybook Design System

Skill de referencia para mantener la coherencia visual de la presentación web de Gossip Garden (Next.js/React/Tailwind). Todo elemento visual debe adherirse a esta estética **cálida, artesanal, infantil y juguetona**. Esta skill se basa en los pilares visuales de la app (Flutter) y las reglas avanzadas de UX/UI.

## Filosofía Visual (Crayon Storybook)

El lenguaje visual combina influencias de libros infantiles y dibujos a crayón. El resultado es **cálido, artesanal y deliberadamente imperfecto**.

1. **La imperfección es intencional:** Evita líneas perfectamente rectas o diseños estériles. Busca formas amigables, orgánicas y redondeadas.
2. **El papel existe:** El fondo no es solo blanco inerte; es textura crema (papel kraft). En la presentación, `globals.css` ya proporciona un fondo radial con textura en la clase `.paper-bg`.
3. **Cero iconos aburridos y NUNCA usar emojis:** *NUNCA* se deben de usar emojis bajo ninguna circunstancia. Si requieres iconografía, usa la librería `lucide-react` o recursos SVG provistos, manteniendo grosores consistentes.
4. **El marrón oscuro (`border-border`, `text-foreground`) une todo:** Es el color principal para textos y contornos (en lugar de negro puro o gris), dándole cohesión de ilustración infantil.

## Paleta de Colores y Tipografía (Tailwind v4)

La presentación ya define un tema de Tailwind en `globals.css`. **Nunca uses colores hex crudos o colores base de Tailwind (`bg-white`, `text-black`, `bg-blue-500`) en los componentes.**

### Colores Base Disponibles:
- **Papel (Fondo global):** `bg-background` (Crema)
- **Tinta (Textos y contornos):** `text-foreground`, `border-border` (Marrón cálido)
- **Superficies de tarjetas:** `bg-card` (Crema claro)
- **Acentos y Botones:** `bg-primary` (Verde hoja/Sprout), `bg-secondary` (Naranja Kraft), `bg-destructive` (Rojo corazón), `bg-accent` (Morado ciruela).
- **Colores Extendidos:** `text-leaf`, `bg-heart`, `border-soil`, etc.

### Sombras:
Nunca uses sombras negras puras (`shadow-md` por defecto). Prefiere sombras teñidas de marrón (`shadow-sm` ya adaptada o la sombra en `.crayon-card`).

### Tipografía:
- **Títulos (Headings):** Usa la fuente Quicksand (`font-heading`). Letras redondeadas, amigables, para títulos, números grandes o métricas.
- **Cuerpo (Body):** Usa Nunito (`font-sans`). Legible pero suave.
- **Estilos a mano:** Usa la clase `.crayon-text` o el seudoelemento de subrayado `.crayon-underline` para destacar palabras importantes como si estuvieran marcadas con crayón.

## Componentes UI (React/Tailwind)

### Tarjetas (Cards)
- Usa la clase global `.crayon-card` para cualquier contenedor o tarjeta principal. Ésta aplica automáticamente el fondo, borde grueso marrón (`3px`), bordes redondeados (`var(--radius-xl)`), sombra cálida, y un filtro especial (`.crayon-outline`) que simula bordes irregulares.
- Si construyes a medida, usa `bg-card border-[3px] border-border rounded-2xl` o `rounded-3xl`.

### Botones (Buttons)
- Formas de píldora (`rounded-full`) o muy redondeados.
- Sin elevación por defecto. Apóyate en colores vibrantes (`bg-primary`, `bg-heart`).
- Usa pseudo-clases simples para interactividad: `hover:scale-105 active:scale-95 transition-transform duration-200`.

### Animaciones Lúdicas (Playful Motion)
- **Clases predefinidas en CSS:**
  - `.breathe`: Animación de respiración suave (escalado sutil continuo). Ideal para ilustraciones hero.
  - `.sway`: Balanceo desde la base. Para plantas o personajes.
  - `.wiggle`: Rotación rápida de rebote, usada cuando un dato resalta o se actualiza.
  - `.bubble-pop`: Para modales o tooltips apareciendo.
- Para transiciones de layout complejas, usa la librería `motion` (Framer Motion). Siempre mantén las animaciones *bouncy* (ej. `spring`, `stiffness: 300, damping: 20`) en lugar de `linear`.

## Reglas de Oro (Checklist Mental)
Antes de construir o modificar componentes en la presentación web, pregúntate:
1. ☐ ¿NUNCA he usado emojis en el código ni en el texto visual?
2. ☐ ¿Estoy usando `lucide-react` para iconos en lugar de caracteres emoji?
3. ☐ ¿Estoy usando el fondo crema (`bg-background`/`bg-card`) y textos marrones (`text-foreground`) en lugar de blanco/negro?
4. ☐ ¿Los bordes de contenedores son ultra redondeados (`rounded-2xl`, `rounded-full`) y, de tener contorno, usan `border-border` grueso?
5. ☐ ¿Las interacciones y elementos importantes tienen micro-animaciones lúdicas (`.breathe`, `.wiggle` o `hover:scale-105`)?
6. ☐ ¿Se siente como un diseño de un libro infantil y mágico que coincide con Gossip Garden?
