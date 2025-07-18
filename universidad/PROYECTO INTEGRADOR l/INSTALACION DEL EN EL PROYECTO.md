# Frontend: React + Vite + TypeScript + Tailwind

## 1. Crear el proyecto React

```bash
mkdir client
cd client
npm create vite@latest . -- --template react-ts
npm install
```

## 2. Instalar Tailwind CSS
npm install tailwindcss @tailwindcss/vite
luego en vite.config.ts 
se añade esto: 
import tailwindcss from '@tailwindcss/vite'

y en plugins: 
plugins: [react(), tailwindcss()],



## 3. Instalar axios para peticiones HTTP

```bash
npm install axios
```
### **¿Para qué sirve Axios?**

Axios sirve para **hacer peticiones HTTP** desde tu frontend o backend en JavaScript.  
Con Axios puedes:

- Obtener datos de un servidor (GET)
    
- Enviar datos (POST)
    
- Actualizar datos (PUT / PATCH)
    
- Eliminar datos (DELETE)

ejemplo: 
import axios from 'axios';

axios.get('https://jsonplaceholder.typicode.com/posts')
  .then(res => {
    console.log(res.data); // Muestra los datos recibidos
  })
  .catch(err => {
    console.error(err); // Maneja errores
  });


## 4. Crear ejemplo básico de login

**src/App.tsx:**

```typescript
import { useState } from 'react';
import axios from 'axios';

function App() {
  const [token, setToken] = useState<string | null>(null);
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      const response = await axios.post('http://localhost:3000/api/login', {
        username,
        password
      });
      setToken(response.data.token);
      alert('Login exitoso!');
    } catch (error) {
      alert('Error en login');
    }
  };

  return (
    <div className="min-h-screen bg-gray-100 flex items-center justify-center">
      <div className="bg-white p-8 rounded-lg shadow-md w-96">
        <h1 className="text-2xl font-bold mb-6 text-center">Login</h1>
        
        {!token ? (
          <form onSubmit={handleLogin} className="space-y-4">
            <input
              type="text"
              placeholder="Username"
              value={username}
              onChange={(e) => setUsername(e.target.value)}
              className="w-full p-2 border rounded"
            />
            <input
              type="password"
              placeholder="Password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              className="w-full p-2 border rounded"
            />
            <button
              type="submit"
              className="w-full bg-blue-500 text-white p-2 rounded hover:bg-blue-600"
            >
              Login
            </button>
          </form>
        ) : (
          <div className="text-center">
            <p className="text-green-600 mb-4">¡Logueado exitosamente!</p>
            <p className="text-sm text-gray-600">Token: {token.substring(0, 20)}...</p>
          </div>
        )}
      </div>
    </div>
  );
}

export default App;
```

## 5. Iniciar el frontend

```bash
npm run dev
```

**URL**: `http://localhost:5173`

---

# Backend: Express + TypeScript + JWT

## 1. Crear el proyecto del servidor

```bash
mkdir server
cd server
npm init -y
```

## 2. Instalar dependencias

```bash
# Dependencias principales
npm install express jsonwebtoken cors

# Dependencias de desarrollo
npm install -D @types/express @types/jsonwebtoken @types/cors @types/node nodemon ts-node typescript
```

## 3. Configurar scripts en package.json

**Agregar a package.json:**

```json
{
  "scripts": {
    "dev": "nodemon src/index.ts",
    "build": "tsc"
  }
}
```

## 4. Configurar TypeScript

```bash
npx tsc --init
```

**Editar tsconfig.json (solo estas líneas importantes):**
borrar y pegar

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  }
}
```

## 5. Crear el servidor básico

**src/index.ts:**

peramente de ejemplo: 
```typescript
import express from 'express';
import cors from 'cors';
import jwt from 'jsonwebtoken';

const app = express();
const PORT = 3000;
const JWT_SECRET = 'mi-secreto-super-seguro';

// Middleware
/*
¿Qué son los Middlewares?

 En Express es una función que se ejecuta entre la petición del cliente y la respuesta del servidor.

Sirven para:

- Procesar datos de la petición
    
- Verificar permisos o autenticación
    
- Manejar errores
    
- Ejecutar lógica antes de enviar la respuesta
 
Funcionamiento básico
Cada vez que un cliente hace una petición, Express pasa por los middlewares en orden, como si fueran filtros.
ejemplo:
app.use((req, res, next) => {
  console.log('Middleware ejecutándose');
  next(); // Sigue al siguiente middleware o ruta
});

¿Qué hace `next()`?

Llama al siguiente middleware o a la función de la ruta.   
Si no se llama a `next()`, la petición se queda atascada.
ejemplo:
// Middleware para registrar cada petición
app.use((req, res, next) => {
  console.log(`Petición recibida: ${req.method} ${req.url}`);
  next();
});

 */
-
app.use(cors());
app.use(express.json());

// Ruta de prueba
app.get('/api/test', (req, res) => {
  res.json({ message: 'Servidor funcionando correctamente!' });
});

// Ruta de login
app.post('/api/login', (req, res) => {
  const { username, password } = req.body;
  
  // Validación básica (aquí iría a la base de datos)
  if (username === 'admin' && password === '123') {
    const token = jwt.sign(
      { username, userId: 1 }, 
      JWT_SECRET,
      { expiresIn: '24h' }
    );
    res.json({ 
      token,
      user: { username, userId: 1 }
    });
  } else {
    res.status(401).json({ error: 'Credenciales incorrectas' });
  }
});

// Middleware para verificar token
const verifyToken = (req: any, res: any, next: any) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Token no proporcionado' });
  }
  
  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Token inválido' });
  }
};

// Ruta protegida de ejemplo
app.get('/api/profile', verifyToken, (req: any, res) => {
  res.json({ 
    message: 'Perfil del usuario',
    user: req.user
  });
});

app.listen(PORT, () => {
  console.log(`Servidor corriendo en puerto ${PORT}`);
});
```

## 6. Iniciar el backend

```bash
npm run dev
```

**URL**: `http://localhost:3000`

## 7. Probar las rutas

- **GET** `http://localhost:3000/api/test` - Ruta de prueba
- **POST** `http://localhost:3000/api/login` - Login (admin/123)
- **GET** `http://localhost:3000/api/profile` - Ruta protegida (necesita token)

---

# Para usar ambos

1. **Terminal 1**: `cd server && npm run dev`
2. **Terminal 2**: `cd client && npm run dev`
3. **Navegar**: `http://localhost:5173`
4. **Login**: usuario `admin`, contraseña `123`

### **¿Para qué sirve JWT y los tokens?**

#### **JWT (JSON Web Token)** es un **método para autenticar usuarios** en aplicaciones web y móviles.

---

### **¿Qué es un token?**

Un **token** es un **pedazo de texto cifrado** que el servidor entrega al usuario después de iniciar sesión.  
Ese token es como un **pase de acceso**: lo guarda el cliente (por ejemplo, en el navegador) y lo envía en cada petición.

---

### **¿Para qué sirve?**

- **Autenticación sin sesiones**  
    El servidor **no guarda el estado del usuario** (no necesita sesiones tradicionales).
    
- **Verificación de identidad**  
    Cada vez que el cliente hace una petición, envía el token y el servidor verifica si es válido.
    
- **Protección de rutas**  
    Permite bloquear o proteger rutas que solo un usuario autenticado puede usar.
    

---

### **¿Cómo funciona JWT?**

1️⃣ **Login:**  
El usuario se loguea → el servidor responde con un **JWT**.

2️⃣ **Petición protegida:**  
El usuario envía el token en el header:

makefile

Copiar código

`Authorization: Bearer TU_TOKEN`

3️⃣ **El servidor verifica el token**  
Si es válido → el usuario puede acceder.  
Si es inválido → devuelve **401 Unauthorized**.

---

### **¿Qué contiene un JWT?**

Un token JWT tiene 3 partes:

css

Copiar código

`HEADER.PAYLOAD.SIGNATURE`

- **Header:** tipo de token y algoritmo de firma
    
- **Payload:** datos del usuario (por ejemplo: id, nombre, rol)
    
- **Signature:** firma digital usando tu clave secreta (para evitar falsificaciones)
    

---

### **Ejemplo visual:**

json

Copiar código

`{   "userId": 1,   "username": "admin",   "exp": 1717800000 }`

Estos datos están **dentro del token**, pero no se pueden modificar sin invalidar la firma.

---

### **¿Por qué usar JWT?**

|Ventajas|Descripción|
|---|---|
|Sin sesiones|No guarda datos del usuario en el servidor|
|Portable|Se puede usar en frontend, backend, APIs, apps móviles|
|Seguro|Si se usa correctamente con HTTPS|
|Rápido|El servidor solo verifica el token, no consulta base de datos|