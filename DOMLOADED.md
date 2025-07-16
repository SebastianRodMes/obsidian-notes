Para asegurarte de que el `btn.addEventListener` se ejecute **solo después de que el DOM esté completamente cargado**, puedes usar el evento `DOMContentLoaded`. Este evento se dispara cuando el documento HTML ha sido completamente cargado y parseado, sin esperar a que se carguen hojas de estilo, imágenes o subframes.

Aquí está cómo se hace:

javascript

CopiarEditar

`document.addEventListener('DOMContentLoaded', () => {     const btn = document.getElementById('btn'); // o usa querySelector     btn.addEventListener('click', () => {         window.location.href = 'index.html';     }); });`