## Descripción
En este tutorial se aprenderá a crear un orquestador que reciba y procese mensajes de Whatsapp utilizando la API con el Sandbox de 360Dialog.

## Creación del servidor
1. Crearemos el servidor de Node siguiendo los pasos del tutorial "servidorNode.md"
2. Lo siguiente será crear el endpoint a donde llegarán las peticiones post de los mensajes (orquestador). Para ello podemos utilizar el siguiente trozo de código
~~~
app.post("/orchestrator", async function (req, res) {
  res.send("OK")

})
~~~
3. Podemos probar que funciona correctamente haciendo una llamada POST con Postman y confirmando que nos devuelve el mensaje de "OK".
4. Ahora ya podemos empezar a configurar las APIs de mensajería que vamos a utilizar (Whatsapp, Telegram, etc). Para este ejemplo utilizaremos la API de Whatsapp que haciendo uso de el entorno de Sandbox de 360Dialog. Lo primero que haremos será solicitar una API Key a 360dialog. Para ello debemos escribirle al contacto de sandbox (+49 30 609859535) por whatsapp el mensaje "START". Devolverá un mensaje con la API Key que sólo podrá ser utilizada para enviar mensajes al número de telefóno con el que solicitaste la API Key.
![image](https://user-images.githubusercontent.com/78566105/217484200-745b01e3-e471-4a43-a3fb-2a8113589491.png)
6. Lo siguiente será indicarle a 360dialog cual es el endpoint del orquestador. 360dialog solo responde a peticiones por https así que será necesario crear un tunel con ngrok para realizar las pruebas. Ejecutaremos el comando `ngrok http puerto` en donde "puerto" será el puerto en donde tenemos corriendo el servidor (EJ: 5000). Ngrok entonces nos proporcionará un enlace https que podemos utlizar indefinidamente hasta que se termine de ejecutar el servidor ngrok.
![image](https://user-images.githubusercontent.com/78566105/217484355-558d4482-6759-4fa7-bedc-976e98b239ec.png)
8. Ahora que tenemos una url de nuestro orquestador válida, el siguiente paso será indicárselo a 360dialog. Podemos usar Postman para ello. Aquí debemos indicarle que será una petición POST a la siguiente url: https://waba-sandbox.360dialog.io/v1/configs/webhook . Como Header debemos indicarle la API que nos proporcionó 360Dialog 
image.png
Y en el body los siguientes parámetros.
~~~
{
"url": "https://a8ba-81-47-173-9.ngrok.io/webhook",
"headers": {
    "Content-Type": "application/json",
    "D360-API-KEY": "6RLENz_sandbox"
    }
}
})
~~~
En donde como se puede observar, se le indica el endpoint https del orquestador en "url" y el apikey. Si la petición se ha realizado con éxito, deberé mostrarse un mensaje como el siguiente:
![image](https://user-images.githubusercontent.com/78566105/217484507-95d90ce3-8dab-4200-a1b2-373787ac1bc4.png)
7. Ahora deberemos indicarle al orquestador cómo recibir mensajes vía Whatsapp. Los mensajes entrarán en el orquestador en formato json, configurado en nuestro enpoint como "req". Podemos extraer de aquí distintos tipos de datos. En este ejemplo veremos cómo se obtner el cuerpo de un mensaje de Whatsapp, el tipo de mensaje y el emisor:
~~~
let message = req.body.messages[0].message
let type = req.body.messages[0].type
let session = req.body.messages[0].from
~~~
8. Ya que podemos conocer ya, el número de teléfono al que se va a responder desde el orquestador, empezaremos a configurar las funciones que se encargarán de enviar los mensajes de respuesta. Para ello podemos añadir la siguiente línea de código:
~~~
whatsapp.sendMessageToWhatsapp(phone, message)
~~~
En donde "whatsapp" es la clase en donde se encuentra el método y "sendMessageToWhatsapp()" es el método que enviará el mensaje, al que se le pasa como parámetros el número de teléfono del receptor y el mensaje. A continuación deberemos de crear el archivo de la clase. Podemos crearla como whatsappAPI.js. En ella introduciremos el siguiente código.
~~~
const axios = require('axios');

async function sendMessageToWhatsapp(phone, response) {
  try {
    let payload = await axios.post(
      "https://waba-sandbox.360dialog.io/v1/messages",
      {
        recipient_type: "individual",
        to: phone,
        type: "text",
        text: {
          body: response,
        },
      },
      {
        headers: {
          "D360-API-KEY": "rMmkWo_sandbox",
        },
      }
    );
    return payload.data;
  } catch (error) {
    console.log(error);
  }
}

module.exports = {
  sendMessageToWhatsapp,
};
~~~
Aqui vemos que se crea un método para el envío de los mensajes por whatsapp por método POST con axios. Dentro de la petición especificamos la url del sandbox a utilizar. En el campo "to", el teléfono del receptor y en "body", el mensaje. También se indica la API key en el header de la petición. Por último se exporta el módulo para que pueda utilizarse el método en otras clases.
9. A continuación, podemos probar que nuestro orquestador envía mensajes vía Whatsapp. Para ello antes debemos importar el módulo mencionado anteriormente:
~~~
const whatsapp = require("./whatsappAPI");
~~~
10. Para finalizar, probar que funciona correctamente la entrada de mensajes en el orquestador, y el envío de mensajes vía Whatsapp.
