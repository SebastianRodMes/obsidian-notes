Para correr codigo con node, en la terminal
node + nombre del archivo que se quiere correr

 Buscar toggle block comment para comentar algo en especifico

funcion flecha con return en una sola linea
const showUserInfo = (username, age ) =>   `The username ${username}, is ${age} old.`

para tener el wordWrap en mode automatico, seria f1, settings json, open user json settings y escribir : "editor.wordWrap": "on",

alt shift a para comentar seleccionando cosas
control shift i
node vs browser
en el browser tenemos cosas como window y document, que en node js no existen, ya que el proposito de node es poder interactuar mas con el sistema operativo, como os.userInfo()

# Global Objects (funciona en node y en browser)
console.log(__dirname) 
resultado:
PS C:\Users\sr331\OneDrive\Escritorio\node-curse> node globals.js 
C:\Users\sr331\OneDrive\Escritorio\node-curse

ruta completa
console.log(__filename)
resultado: 
C:\Users\sr331\OneDrive\Escritorio\node-curse\globals.js


console.log(module)
{
  id: '.',
  path: 'C:\\Users\\sr331\\OneDrive\\Escritorio\\node-curse',
  exports: {},
  filename: 'C:\\Users\\sr331\\OneDrive\\Escritorio\\node-curse\\globals.js',
  loaded: false,
  children: [],
  paths: [
    'C:\\Users\\sr331\\OneDrive\\Escritorio\\node-curse\\node_modules',
    'C:\\Users\\sr331\\OneDrive\\Escritorio\\node_modules',
    'C:\\Users\\sr331\\OneDrive\\node_modules',
    'C:\\Users\\sr331\\node_modules',
    'C:\\Users\\node_modules',
    'C:\\node_modules'
  ],
  [Symbol(kIsMainSymbol)]: true,
  [Symbol(kIsCachedByESMLoader)]: false,
  [Symbol(kIsExecuting)]: true
}

# Timers
el interval, ejecuta lo mismo repetitiva, cada dos segundos va a imprimir holi
 setInterval(()=> {

    console.log('holi')

 }, 2000)

  
el Timeout, despues de 2 segundos va a imprimir holi2, pero solo lo hará una vez 
  setTimeout(()=> {

    console.log('holi2')

 }, 2000)

# MODULES 

module/myModule
const myWebAddress = "sebas.com"

module.exports = myWebAddress


console.log(module)
resultado:  exports: 'sebas.com',

--------------------------------------------
ahora, si se quiere traer o llamar algo desde ese exports, hay que usar require():
 main.js
const title = require('./module/myModule')

console.log(title)
resultado: sebas.com
Nota: ahi se puede almacenar cualquier tipo de variable y multiples variables.



carpeta math

archivo index.js 
function add (x, y){

    return x + y

}

function subtract (x, y){

    return x - y

}

function multiply (x, y){

    return x * y

}

  

function divide (x, y){

    return x / y

}

module.exports = {

    add,

    subtract,

    multiply,

    divide

}
esto lo guarda digamos en module y luego aca en 
main.js 
const math = require('./math/index')

console.log(math.add(2,2)) accedemos a el 

# MODULO OS, para el manejo de cosos del sistema operativo. 

const os = require('os')

/* console.log(os.userInfo())

console.log(os.uptime())

console.log(os.platform())

console.log(os.totalmem())

console.log(os.freemem()) */

console.table({

    os: os.platform(),

    version: os.release(),

    totalMemory: os.totalmem()

  

})