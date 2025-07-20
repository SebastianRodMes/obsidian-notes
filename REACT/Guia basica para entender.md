Instalacion: 
C:\Users\sr331\OneDrive\Escritorio>npx create-vite@latest react01
o Select a framework: | React
o Select a variant: | TypeScript
C:\Users\sr331\OneDrive\Escritorio> cd react01 C:\Users\sr331\OneDrive\Escritorio\react01>npm install

C:\Users\sr331\OneDrive\Escritorio\react01>code .

## Primero, entiender qué tengo

El proyecto tiene esta estructura:

```
react01/
├── src/
│   ├── App.tsx       // Tu componente principal
│   ├── main.tsx      // Punto de entrada (como index.js)
│   └── App.css       // Estilos
├── index.html        // Página base
└── package.json      // Configuración
```

**`src/App.tsx` y borrar todo, pon esto:**

tsx

```tsx
function App() {
  return (
    <div>
      <h1>¡Hola, soy tu primera app!</h1>
      <p>Esto funciona con React</p>
    </div>
  );
}

export default App;
```

## Ejercicio 2: Usar variables

**Cambia `App.tsx` por esto:**

tsx

```tsx
function App() {

  const miNombre = "Sebastian";

  const miEdad = 25;

  const meGustaReact = false;

  

  return (

    <div>

      <h1>Hola, soy {miNombre}</h1>

      <p>Tengo {miEdad}</p>

      <p>Me gusta react?{meGustaReact ? "Si":"NO"}</p>

    </div>

  );

}

export default App;
```


Boton interactivo:
import {useState} from 'react';

function App(){

  const [contador, setContador] = useState(0);

  

  return (

<div>

  <h1>Contador: {contador}</h1>

  <button onClick={() => setContador(contador +1)}>Sumar 1</button>

  <button onClick={() => setContador(contador +2)}>Sumar 2</button>

  <button onClick={()=> setContador(0)}>Reset</button>

</div>

  );

}

  

export default App;

- `useState(0)` crea una variable que empieza en 0
- `contador` es el valor actual
- `setContador` es la función para cambiar el valor
- Cuando haces clic, cambia el número


## Ejercicio 4: Input que cambia texto

tsx

```tsx
import { useState } from 'react';

function App() {
  const [texto, setTexto] = useState("Escribe algo...");
  
  return (
    <div>
      <h1>{texto}</h1>
      <input 
        value={texto}
        onChange={(e) => setTexto(e.target.value)}
        placeholder="Escribe aquí"
      />
      <button onClick={() => setTexto("")}>
        Limpiar
      </button>
    </div>
  );
}

export default App;
```

**Explicación:**

- `e.target.value` es lo que escribiste en el input
- `onChange` se ejecuta cada vez que escribes algo

## Ejercicio 5: Lista de tareas básica

tsx

```tsx
import { useState } from 'react';

function App() {
  const [tareas, setTareas] = useState(["Aprender React", "Hacer ejercicio"]);
  const [nuevaTarea, setNuevaTarea] = useState("");
  
  const agregarTarea = () => {
    if (nuevaTarea.trim() !== "") {
      setTareas([...tareas, nuevaTarea]);
      setNuevaTarea("");
    }
  };
  
  return (
    <div>
      <h1>Mi Lista de Tareas</h1>
      
      <input 
        value={nuevaTarea}
        onChange={(e) => setNuevaTarea(e.target.value)}
        placeholder="Nueva tarea"
      />
      <button onClick={agregarTarea}>
        Agregar
      </button>
      
      <ul>
        {tareas.map((tarea, index) => (
          <li key={index}>{tarea}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

**¿Qué hace esto?**

- `tareas` es un array que empieza con 2 tareas
- `[...tareas, nuevaTarea]` agrega la nueva tarea al final
- `map()` convierte cada tarea en un `<li>`
- `key={index}` es obligatorio para listas


### ¿Qué es `e.target.value`?

- `e.target` → Es el **elemento HTML que disparó el evento** (en este caso, el `<input>`).
    
- `e.target.value` → Es el **valor actual del input**, o sea, lo que el usuario escribió.