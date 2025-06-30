# Prototipado de ParkCar

En este repositorio se almacena el prototipo de la aplicación móvil **ParkCar**. Para fines prácticos y considerando la familiaridad de los estudiantes participantes con el framework, este prototipo fue desarrollado utilizando Vue.js 3 junto con Vite como herramienta de construcción.

El prototipo fue adaptado para ser compatible tanto con computadoras como con dispositivos móviles, incluyendo teléfonos celulares y tablets, permitiendo su visualización y uso en distintos entornos.

## Guía para ejecutar el prototipo de ParkCar

### 1. Abrir una terminal en Visual Studio Code

Dirígete a la pestaña **Terminal** en la parte superior y selecciona **New Terminal**, o simplemente presiona el atajo de teclado:

Ctrl + Shift + Ñ

Esto abrirá la terminal integrada en Visual Studio Code.

### 2. Instalar dependencias del proyecto

Ejecuta el siguiente comando para instalar todas las dependencias listadas en el archivo `package.json`:

```
npm install
```

### 3. **(Opcional)** Modificar la dirección IP del host para habilitar la transmisión en la red.

Si deseas probar el prototipo en una red privada, debes realizar algunas modificaciones en los archivos de configuración.

Primero, abre el archivo:

- `vite.config.js`

Dentro de este archivo, localiza la configuración de **host** y reemplaza su valor por la dirección IP de tu equipo.

Luego, realiza el mismo cambio en el archivo:

- `package.json`

Modifica el valor de la variable **host** por la IP de tu computador.

> ⚠️ Asegúrate de que los dispositivos estén conectados a la misma red.

### 4. Ejecutar el prototipo

Para ejecutar el prototipo de forma local en tu equipo, utiliza el siguiente comando:

```
npm run dev
```

Si deseas ejecutar el prototipo dentro de una red privada, primero debes realizar el **paso 3**, y luego ejecutar:

```
npm run host
```

## Notas adicionales

- Este prototipo es de uso académico y fue adaptado por los estudiantes participantes del proyecto.
- Si tienes problemas de conexión en red, verifica que tu firewall o antivirus no bloquee las conexiones entrantes.