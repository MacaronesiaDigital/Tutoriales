## Descripción
En este tutorial se mostrará cómo crear un webhook para Dialogflow con NodeJS y Express. También se explicará el tratamiento de los datos de Dialogflow desde este webhook.

### Notas
Para la realización de este tutorial será recomendable realizar antes el tutorial de [cómo crear un servidor en NodeJS](https://github.com/OliverAlamoHolm)

##  Creación del Webhook
1. Una vez creado el servidor de NodeJS e instalado todas las dependencias, se añadirá al código el endpoint del webhook de DialogFlow, al que se accederá por un método POST

~~~
app.post("/webhook", async function (req, res) {

})
~~~

2. Para que DialogFlow pueda trabajar con un webhook, deberemos indicarselo. Para esto nos situamos en la sección de "Fulfillment" y activaremos el slot.  

![image](https://user-images.githubusercontent.com/78566105/205516571-959e3a49-7916-4ed1-900a-0e14f0a1a433.png)

3. Ahora podremos indicar qué intenciones pueden ser tratadas desde el webhook. Para ello podemos acceder a la intención de Dialogflow, y en la parte inferior de la pantalla, activaremos el slot de uso de fulfillment también:

![image](https://user-images.githubusercontent.com/78566105/205516709-e464352d-4c3b-48fc-a9c3-3dfd0f987b1b.png)

4. Ahora debemos indicarle a Dialogflow cuál va a ser nuestro webhook, para ello necesitaremos que el servidor esté ejecutándose de manera remota y sea accesible desde el exterior. Podemos utilizar ngrok gratuitamente para ello, por lo que lo instalamos desde su [página web](https://ngrok.com/). Sería recomendable registrarse y pedir un authtoken, ya que esto nos dará la ventaja de utilizar ngrok indefinidamente. Una vez obtenido el authtoken, ejecutaríamos este comando:  `ngrok config add-authtoken <token>`  
Una vez instalado el ngrok, montamos el túnel con el siguiente comando: `ngrok http <puerto>` indicándole el puerto en el que tenemos ejecutando el servidor en local. Ngrok proporcionará un enlace tunel https.

![image](https://user-images.githubusercontent.com/78566105/205517179-b3c3be66-82f3-4ccf-9e37-92d8e4059348.png)

5. Ahora podemos indicarle a Dialogflow dónde estará nuestro webhook. Para ello iremos a la sección de "fulfillmient" y le daremos nuestro enlace de ngrok junto con nuestro endpoint en el input de url. -	Como podemos ver en la imagen, también podemos añadir headers y auth para hacer las comunicaciones más seguras.

![image](https://user-images.githubusercontent.com/78566105/205517250-e6be8add-22e1-410d-8564-e78bf9e33955.png)

6. El siguiente paso sería configurar nuestro webhook para que podamos recibir intenciones de dialogflow. Para ello deberíamos instalar la librería de dialogflow en nuestro servidor Node con: `npm i dialogflow` y `npm i dialogflow-fullfilment` y señalar el uso de estas librerías en el código
~~~
const dfff = require('dialogflow-fulfillment')
~~~

7. Ahora configuraremos el endpoint del webhook. Necesitamos determinar la respuesta de Dialogflow en la que nos enviará toda la información sobre la intención. A esto lo llamaremos agente.
~~~
var url = req.headers.host + '/' + req.url;
const agent = new dfff.WebhookClient({
    request: req,
    response: res
});
~~~

8. Ahora indicaremos cuáles serán las intenciones de Dialogflow que se tratarán desde nuestro webhook, para ello utilizaremos el método intent.setMap() e indicaremos el nombre de la intención de DialogFlow, y el nombre de la función del webhook que la procesará. Aquí se muestra un ejemplo de uso para dos intenciones: 
~~~
var intentMap = new Map();
intentMap.set('demo', demo);
intentMap.set('deleteUser', deleteUserTrigger)
agent.handleRequest(intentMap);
~~~

9. Una vez hecho esto, podremos manipular los datos de la intención(que es un json) dentro de la función y enviarle una respuesta a Dialogflow con la función agent.add(mensaje):  
~~~
async function demo(agent) {
  agent.add("...")
}
~~~

##  Manejo de datos de Dialogflow desde Webhook
Ya se ha indicado cómo crear un webhook para un agente de Dialogflow y tratar las distintas intenciones en funciones Javascript. A continuación se explicarán los distintos datos de dailogflow que podemos procesar desde las funciones.

### Entidades
Si una de nuestras intenciones maneja una o varias entidades en ellas, podemos extraerla desde el agente del campo "parameters" de la siguiente manera  
`var ubication = agent.context.get('getcropdrs-location-followup').parameters.location`  
Aquí lo que queremos hacer es extraer una entidad de localización de la intención, para ello se ha accedido al campo de contexto del agente, se ha indicado el contexto de la entidad que queremos obtener(siempre en minúscula)y se ha accedido al campo de parámetros en donde luego se le ha indicado que se quiere la localización. Si este parámetro no existira, la variable tendría un valor de **null** o **undefined**.
Para entidades con nombres compuestos, se tiene que indicar el nombre de la entidad entre corchado y entre comillado.  
`var userName = agent.context.get('getusername-followup').parameters['given-name']`

### Sesión
Podemos obtner la sessión de usuario del remitente del mensaje de la siguiente manera:
`var session = req.body.session`

