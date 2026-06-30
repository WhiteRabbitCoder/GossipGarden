---
name: flutter-storybook-design
description: Crayon Storybook Design System adapted for Flutter. Enforces the paper-like, playful, and warm aesthetic of Gossip Garden for all UI elements. Trigger whenever creating or styling Flutter UI components, screens, or animations.
---

# Gossip Garden — Flutter Storybook Design System

Skill de referencia para mantener la coherencia visual del proyecto Gossip Garden en Flutter (app móvil). Todo elemento visual debe adherirse a esta estética **cálida, artesanal, infantil y juguetona**.

## Filosofía Visual (Crayon Storybook)

El lenguaje visual combina influencias de libros infantiles y dibujos a crayón. El resultado es **cálido, artesanal y deliberadamente imperfecto**.

1. **La imperfección es intencional:** Evita líneas perfectamente rectas o diseños estériles de Material Design por defecto. Busca formas amigables y orgánicas.
2. **Todo tiene cara:** Los personajes (las plantas) son expresivos y tienen emociones reales.
3. **El papel existe:** El fondo no es solo blanco inerte; es textura crema (papel kraft).
4. **Cero iconos aburridos y NUNCA usar emojis:** *NUNCA* se deben de usar emojis. Cuando pase eso (se necesite un gráfico o icono), preferiblemente buscar un icono ya creado en el proyecto, o hacer uno mediante una herramienta para crear imágenes con NanoBanana. Evita los iconos afilados estándar del sistema.
5. **El marrón oscuro (`#3D2817`) une todo:** Es el color principal para textos y contornos (en lugar de negro puro o gris), dándole esa cohesión de ilustración infantil.

## Paleta de Colores Base (Adaptación a Flutter)

Define estos colores en tu `AppTheme` o como constantes globales. **Nunca uses colores hex o de Material "sueltos" en los widgets si existen aquí.**

- **Cream Paper (Fondo global):** `Color(0xFFFAF1DA)`
- **Cream Dark (Alternativo/Hover):** `Color(0xFFF0E2C0)`
- **Ink (Texto principal / Contornos):** `Color(0xFF3D2817)`
- **Ink Soft (Texto secundario):** `Color(0xFF6B4A2E)`
- **Pot Orange (Botones/Acentos neutros):** `Color(0xFFE8A95C)`
- **Leaf Green (Acentos vegetales):** `Color(0xFF8AC553)`
- **Heart Red (CTA Principal / Rubor):** `Color(0xFFE85D52)`
- **Cream Light (Fondo de tarjetas):** `Color(0xFFFFF8E7)`

*Regla de Sombras*: Siempre usa colores basados en `Ink` con baja opacidad. **Nunca negro puro (`Colors.black`).** Ej: `Color(0xFF3D2817).withOpacity(0.08)`.

## Tipografía (Adaptación a Flutter)

Usa el paquete `google_fonts` (o equivalentes locales).

- **Headings (Títulos, Logos, Precios):** `Quicksand` (Pesos gruesos: w700, w800, w900). Letras redondeadas y muy amigables.
- **Body (Cuerpo de texto, UI general):** `Nunito` (Pesos: w400, w600, w700). Legible pero suave.
- **Decorativa (Expresiones cortas):** `Caveat` (Pesos: w500, w600, w700). Solo para destacar o imitar escritura a mano, nunca en párrafos largos.

## Componentes UI (Flutter)

### Tarjetas (Cards y Contenedores)
- Evita el widget `Card` de Material puro con su elevación por defecto.
- Usa `Container` con `BoxDecoration`:
  - Color de fondo: `Cream Light` (`Color(0xFFFFF8E7)`).
  - Bordes muy redondeados (`BorderRadius.circular(24)`).
  - Si necesitas delinear, usa un `Border` de color `Ink` (`Color(0xFF3D2817)`) de unos `2.0` o `3.0` de grosor, simulando el borde fuerte de un dibujo.

### Botones (Buttons)
- Usa formas de pastilla o píldora (`StadiumBorder` o `RoundedRectangleBorder` con radius enorme).
- Nada de sombras duras estándar. Preferiblemente sin elevación (`elevation: 0`), apoyándose en el color de fondo vibrante (`Heart Red` o `Leaf Green`) y texto en `Cream Paper`.
- Al presionarse, busca micro-interacciones que den una sensación de peso o de hundimiento.

### Animaciones Lúdicas (Playful Motion)
- **Curvas de Easing:** Huye del lineal. Usa curvas suaves como `Curves.easeInOutCubic`, `Curves.easeOutBack` o `Curves.elasticOut` muy sutiles para dar una sensación "bouncy" o de rebote al abrir modales o mostrar nuevos elementos.
- **Micro-animaciones:** Elementos que flotan suavemente (ej. un `TweenAnimationBuilder` con un *offset* repetitivo), bocadillos de diálogo que se balancean sutilmente.

## Reglas de Oro (Checklist Mental)
Antes de construir o maquetar en Flutter, pregúntate:
1. ☐ ¿Estoy usando el fondo crema (`Cream Paper`) y textos marrones (`Ink`) en vez de fondo blanco y texto negro?
2. ☐ ¿Los bordes de contenedores y botones son ultra redondeados y sin esquinas afiladas?
3. ☐ ¿Las sombras están teñidas de marrón con baja opacidad en lugar de grises o negros?
4. ☐ ¿La tipografía es Quicksand/Nunito, huyendo del aspecto corporativo o rígido del sistema?
5. ☐ ¿Se siente como un diseño de un libro infantil, amigable y acogedor?
