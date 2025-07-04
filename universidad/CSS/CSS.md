## 📄 Estructura general de una regla CSS

css

CopyEdit

`selector {   propiedad: valor; }`

**Ejemplo:**

css

CopyEdit

`h1 {   color: blue;   font-size: 24px; }`

---

## 📘 1. **Selectores básicos**

|Selector|Ejemplo|Significado|
|---|---|---|
|Elemento|`p {}`|Todos los párrafos|
|Clase|`.mi-clase {}`|Todos los elementos con clase "mi-clase"|
|ID|`#mi-id {}`|Elemento con id "mi-id"|
|Todos|`* {}`|Todos los elementos|
|Hijo directo|`div > p {}`|Solo los `p` dentro de un `div`, no nietos|
|Combinado|`div, p {}`|Afecta a ambos|

---

## 🎨 2. **Propiedades comunes**

### 📐 Tamaño y espacio

- `width` / `height`: ancho / alto.
    
- `margin`: espacio fuera del borde.
    
- `padding`: espacio dentro del borde.
    
- `border`: borde (grosor, tipo, color).
    

css

CopyEdit

`.box {   width: 200px;   height: 100px;   padding: 10px;   margin: 20px;   border: 2px solid black; }`

---

### 🔠 Texto

- `color`: color del texto.
    
- `font-size`: tamaño de letra.
    
- `font-family`: tipo de letra.
    
- `text-align`: alineación (left, center, right).
    
- `font-weight`: negrita (`normal`, `bold`, `400`, `700`, etc.)
    

---

### 🎨 Colores

- Por nombre: `red`, `blue`, `green`.
    
- Hexadecimal: `#ff0000`
    
- RGB: `rgb(255, 0, 0)`
    
- Transparencia: `rgba(0, 0, 0, 0.5)`
    

---

### 🧱 Fondos

- `background-color`: color de fondo.
    
- `background-image`: imagen de fondo.
    
- `background-size`: tamaño (`cover`, `contain`).
    
- `background-position`: posición de la imagen.
    

---

## 📦 3. **Modelo de Caja (Box Model)**

Todo elemento HTML es como una caja:

csharp

CopyEdit

`[margin]   [border]     [padding]       [contenido]`

---

## 🧭 4. **Posicionamiento**

- `static`: por defecto.
    
- `relative`: relativo a su posición original.
    
- `absolute`: respecto a su contenedor padre con `position: relative`.
    
- `fixed`: posición fija (como menús que no se mueven).
    
- `sticky`: se pega al hacer scroll.
    

---

## 🔧 5. **Display y Flexbox**

### 🔲 Display

- `block`: ocupa toda la línea (ej. `div`)
    
- `inline`: dentro del texto (ej. `span`)
    
- `inline-block`: como `inline`, pero acepta `width` y `height`
    
- `none`: no se muestra
    

---

### 🧱 Flexbox (para layouts)

Para organizar elementos horizontal o verticalmente:

css

CopyEdit

`.container {   display: flex;   justify-content: center;     /* alineación horizontal */   align-items: center;         /* alineación vertical */   gap: 10px;                   /* espacio entre hijos */ }`

---

## ✨ 6. **Transiciones y efectos**

css

CopyEdit

`.button {   transition: background-color 0.3s ease; }  .button:hover {   background-color: blue; }`

---

## 📱 7. **Responsivo (media queries)**

css

CopyEdit

`@media (max-width: 600px) {   .container {     flex-direction: column;   } }`

---

## 📂 8. **Cómo escribir CSS**

### a) **Interno (dentro del HTML)**

html

CopyEdit

`<style>   p { color: red; } </style>`

### b) **Externo (archivo `.css`)**

html

CopyEdit

`<link rel="stylesheet" href="styles.css">`

## Guía práctica de unidades en CSS

### ✅ 1. `px` (píxeles)

- **Absoluta**: no cambia según el tamaño del texto o pantalla.
    
- **Uso común**: bordes, imágenes, líneas, tamaños muy precisos.
    
- 📌 **Útil cuando** querés control exacto.
    

css

CopyEdit

`.card {   width: 300px;   border: 2px solid black; }`

---

### ✅ 2. `rem` (root em)

- **Relativa**: basada en el tamaño de fuente raíz del `<html>` (normalmente 16px).
    
- 1rem = 16px (por defecto, si no cambiaste nada).
    
- **Uso ideal**: márgenes, paddings, fuentes que escalen bien.
    

css

CopyEdit

`body {   font-size: 1rem; /* 16px */ }  h1 {   font-size: 2rem; /* 32px */   margin-bottom: 1rem; /* 16px */ }`

---

### ✅ 3. `em`

- **Relativa**: basada en el tamaño de fuente del **elemento padre inmediato**.
    
- Puede **acumularse** si hay varios niveles anidados.
    

css

CopyEdit

`.container {   font-size: 20px; }  .container p {   font-size: 1em; /* = 20px */ }  .container .small {   font-size: 0.8em; /* = 16px */ }`

---

### ✅ 4. `%` (porcentaje)

- **Relativa**: al contenedor padre.
    
- Útil para **layouts fluidos**, o para altura/ancho proporcionales.
    

css

CopyEdit

`img {   width: 100%; /* se adapta al ancho del contenedor */ }`

---

### ✅ 5. `vh` / `vw` (viewport height / width)

- Basado en el tamaño de la ventana del navegador.
    
- 100vh = 100% de la altura visible.
    

css

CopyEdit

`.hero {   height: 100vh; /* toda la altura del navegador */ }`

---

## 🎯 ¿Entonces cuál uso?

|Objetivo|Mejor unidad|
|---|---|
|Tamaño de fuente adaptable|`rem`|
|Precisión exacta|`px`|
|Layouts fluidos|`%`, `vh`, `vw`|
|Elementos relacionados|`em`|

---

### 💡 Recomendación moderna

- Usá **`rem`** para **tipografía, padding y márgenes**.
    
- Usá **`%` o `vh/vw`** para hacer que el diseño sea **responsivo**.
    
- Usá **`px`** solo si querés control fino en cosas pequeñas como bordes o iconos.