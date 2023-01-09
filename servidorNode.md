## Descripción
En este tutorial se aprenderá a crear un servidor NodeJS con Express.

## Creación del servidor
1. Descargaremos e instalaremos NodeJS para así empezar a utilizar su gestor de comandos NPM
2. Creamos la carpeta en donde estableceremos nuestro proyecto, y en su interior ejecutamos `npm init` para inicializar el proyecto Node y crear el fichero package,json
3. Creamos el fichero index.js, fichero .env y .gitignore.
4. Instalamos express, dotenv, http, axios, nodemon con NPM: `npm i express dotenv http axios nodemon`
5. Añadimos todos los requerimientos y declaramos variables necesarias.
~~~
const express = require('express');
const app = express();
const axios = require('axios');
~~~
7. Creamos un endpoint para peticiones GET de prueba.  
~~~
app.get('/prueba', (req, res) =>{
  res.send("Esto es una prueba")
})
~~~

8. Hacemos que el servidor express se ejecute en el puerto que le indicamos.  
~~~
app.listen(process.env.PORT || 5000, function () {
    console.log("Server is live on port 5000")
});
~~~
9. Ejecutamos el servidor con `nodemon index.js` y comprobamos que funciona correctamente haciendo una llamada get al endpoint de prueba desde el navegador.
