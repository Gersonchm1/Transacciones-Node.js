# Transacciones en MongoDB


## ¿Qué son las transacciones?


Las transacciones en MongoDB permiten agrupar múltiples operaciones de lectura/escritura en una única unidad atómica. Esto garantiza que:


- **Todas las operaciones se ejecuten con éxito** (*commit*).
- **O ninguna se aplique** (*rollback*) si falla alguna, manteniendo la consistencia de los datos.


### Características clave


**Atomicidad**: Operaciones "todo o nada". 
**Multi-documento y multi-colección**: Útil para lógicas complejas que involucran varios datos.   
**Control del Rolback**: *Rollback* automático en fallos.


---


## ¿Para qué se usan?


### Casos comunes


1. **Transferencias bancarias**:
  - Debitar de una cuenta **y** acreditar en otra (si falla un paso, se revierte todo).
2. **Pedidos en e-commerce**:
  - Actualizar inventario **+** registrar orden **+** cobrar al cliente.
3. **Sistemas de reservas**:
  - Bloquear asiento **+** generar ticket **+** notificar al usuario.






---


## ¿Por qué son importantes?


1. **Consistencia en operaciones complejas**: Evita estados intermedios corruptos.
2. **Seguridad en aplicaciones críticas**: Ej: fintech, salud, etc.
3. **Flexibilidad**: Permite modelos de datos normalizados sin perder atomicidad.


---


## Ejemplo de código ( Sin Node.js)


```javascript
const session = db.getMongo().startSession(); // Iniciar sesión


session.startTransaction(); // Iniciar transacción


try {
  // Restar $100 a Ana
  db.cuentas.updateOne(
    { nombre: "Ana" },
    { $inc: { saldo: -100 } },
    { session }
  );


  // Sumar $100 a Luis
  db.cuentas.updateOne(
    { nombre: "Luis" },
    { $inc: { saldo: 100 } },
    { session }
  );


  session.commitTransaction(); // Confirmar la transacción
  print("✅ Transacción completada con éxito");


} catch (error) {
  session.abortTransaction(); // Revertir si hubo error
  print("❌ Transacción cancelada por error:", error);
}


session.endSession(); // Finalizar la sesión


```










---


## Ejemplo de código ( Node.js)


```javascript
const { MongoClient } = require("mongodb");


async function main() {
  const client = new MongoClient("mongodb://localhost:27017");


  await client.connect();
  const db = client.db("miBD");
  const cuentas = db.collection("cuentas");
  const session = client.startSession();


 try {
  session.startTransaction();

  await cuentas.updateOne(
    { nombre: "Ana" },
    { $inc: { saldo: -100 } },
    { session }
  );

  await cuentas.updateOne(
    { nombre: "Luis" },
    { $inc: { saldo: 100 } },
    { session }
  );

  await session.commitTransaction();
  console.log("✅ Transacción exitosa");

} catch (error) {
  await session.abortTransaction();  // Aquí se revierten los cambios
  console.error("❌ Transacción falló y fue revertida:", error);

} finally {
  await session.endSession();
  await client.close();
}



main();




```

```javacript
[
  {
    "nombre": "Ana",
    "saldo": 500
  },
  {
    "nombre": "Luis",
    "saldo": 200
  }
]

```
# Ejecutar ejemplo de transacción con MongoDB y Node.js desde CMD

Este instructivo te guía paso a paso para crear y ejecutar un ejemplo de transacción entre dos cuentas usando MongoDB y Node.js, todo desde la terminal CMD.

## Requisitos previos

- Tener Node.js instalado correctamente (`node -v`, `npm -v`).

- Usar terminal CMD (no PowerShell ni VS Code).

## 1. Crear carpeta del proyecto

```
cd %USERPROFILE%\Desktop
mkdir nodejs-mongo
cd nodejs-mongo
```
## 2. Inicializar proyecto Node.js
```

npm init -y
```
## 3. Instalar el driver de MongoDB
```
npm install mongodb
```
## 4. Crear el archivo index.js
Crea el archivo index.js y pega este contenido:

```
const { MongoClient } = require("mongodb");

async function main() {
  const client = new MongoClient("mongodb://localhost:27017");

  await client.connect();
  const db = client.db("miBD");
  const cuentas = db.collection("cuentas");
  const session = client.startSession();

  try {
    session.startTransaction();

    await cuentas.updateOne(
      { nombre: "Ana" },
      { $inc: { saldo: -100 } },
      { session }
    );

    await cuentas.updateOne(
      { nombre: "Luis" },
      { $inc: { saldo: 100 } },
      { session }
    );

    await session.commitTransaction();
    console.log("Transacción exitosa");

  } catch (error) {
    await session.abortTransaction();
    console.error("Transacción fallida y revertida:", error);
  } finally {
    await session.endSession();
    await client.close();
  }
}

main();
```

## 5. Insertar datos iniciales en MongoDB
Abre el shell de MongoDB (mongo) y ejecuta:

```
use miBD

db.cuentas.insertMany([
  { nombre: "Ana", saldo: 500 },
  { nombre: "Luis", saldo: 200 }
])
```
## 6. Ejecutar el código desde CMD
Vuelve a la carpeta del proyecto y corre:

```
node index.js
```
Resultado esperado
En consola verás:
```

Transacción exitosa
```








