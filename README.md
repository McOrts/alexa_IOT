# Alexa te informa de la temperatura y el sol de la Playa de Palma 

No podía esperar para poder usar los comandos de voz de mi Alexa Echo para leer, encender y apagar dispositivos. Y hace un año monté la [estación metereológia móvil basada en el arduino MRKFOX1200](https://github.com/McOrts/MKRFOX1200_mobile-weather-station). Ahora cualquier Alexa del mundo puede informar de la temperatura, presión atmostférica y radiación untravioleta en la Playa de Palma.

  <a href="http://www.youtube.com/watch?feature=player_embedded&v=WW9ZDhAB9yA
  " target="_blank"><img src="http://img.youtube.com/vi/WW9ZDhAB9yA/0.jpg"
  alt="Basic video" width="240" height="180" border="10" /><

## Alexa conectada al IOT

El concepto es conectar un perfil (Skill) a un interface con un dispositivo IOT basado en Arduino para poder interactuar con el mundo físico. Desde encender una bombilla, o leer un sensor de temperatura, hasta controlar un robot remotamente. Y tendremos que empezar por definir un _Invocation Name_ al que Alexa atienda cuando le digamos: Alexa! En mi caso:

Alexa! ask weather station

## Arquitectura

La solución presentada aquí utiliza los servicios web de Amazon (AWS) para hacer toda la interfaz de voz con Alexa y el backend, Things Speak como repositorio para los datos de los sensores del Arduino y la red SigFox para subir los datos a Things Speak. De esta manera tanto el Alexa como el dispositivo IOT pueden estar situados en casi cualquier parte del mundo y mantener su interconectividad.

![Arquitectura alexa ESP8266](https://github.com/McOrts/alexa_IOT/blob/master/images/alexa-esp-ts_architecure.jpg?raw=true)

Al ver el diagrama quizás te hayas preguntado ¿Por qué no comunicarse directamente con el ESP8266 desde la función AWS Lambda? Porque implicaría saltarse varias reglas de seguridad exponiendo las credenciales del dispositivo para que AWS pudiera acceder. Amazon tiene su propia solución para esto, el AWS Greengrass, pero implica un coste y es más complejo. Things Speak actua de cortafuego y nos permite el acceso a los datos desde multiples clientes además de Alexa.

## ¿Qué necesitamos?
Los componentes utilizados para este proyecto son estos:
* ![Amazon Alexa Echo Dot](https://github.com/McOrts/alexa_IOT/blob/master/images/echo_dot.png?raw=true) [Amazon Alexa Echo Dot](http://amzn.eu/d/8Blx0LD)
* ![Alexa Skills Kit](https://github.com/McOrts/alexa_IOT/blob/master/images/alexa_skill.jpg?raw=true) [Alexa Skills Kit](https://developer.amazon.com/alexa/console/ask)
* ![AWS Lambda](https://github.com/McOrts/alexa_IOT/blob/master/images/aws_lambda.jpg?raw=true) [AWS Lambda](http://aws.amazon.com)
* ![ThingSpeak API](https://github.com/McOrts/alexa_IOT/blob/master/images/ThingSpeak.jpg?raw=true) [API de ThingSpeak](https://thingspeak.com)
	
## Montando el diálogo
**Empezaremos con el comando vocal:**
_“Alexa, ask weather station for measures“_

**Alexa respondrá:**
_“the temperature is 13 degrees the index of ultraviolet radiation is -1.2 and the atmospheric pressure is 1016“_

El _Interaction Model_ incluye la configuración necesaria para que el frontal de Alexa  
	1. On the left hand navigation panel, select the **JSON Editor** tab under **Interaction Model**. In the textfield provided, replace any existing code with the code provided in the [Interaction Model](../models/en-US.json).  Click **Save Model**.
    2. If you want to change the skill invocation name, select the **Invocation** tab. Enter a **Skill Invocation Name**. This is the name that your users will need to say to start your skill.  In this case, it's preconfigured to be 'space facts'.
    3. Click "Build Model".

Implementation
We need three components to make this system function as intended.

Data Source (ESP8266 Sensor Data)

From ThingSpeak Channel or
Directly from ESP8266
Amazon Skill to convert your voice to a command to retrieve the requested information

Amazon Lambda function to receive the Skill request, query for the requested information and return a reply to the Amazon Skill

Let’s do it…

You will need to setup two accounts with Amazon, If you do not already have these accounts, create them now. No worries, they are FREE:

Create Amazon Developer Account (For Alexa Skill)

Start by pointing your browser to:

https://developer.amazon.com/

Select “Sign In” located on the page’s top-right header:

Next, select the “Create your Amazon Developer Account” button

Follow the instructions to create your free Amazon Developer Account

Create free AWS account (For Lambda Function)

Start by pointing your browser to:

https://portal.aws.amazon.com/billing/signup#/start

Follow the instructions to create your free AWS (Amazon Web Services) Account

Creating the Alexa Skill

Sign in to https://developer.amazon.com/ and go to the Developer Console.

Select the Alexa tab.

Select “Get Started” Alexa Skills Kit.

Select “Add a New Skill”

Note: The rest of the skill creation can be customized for your needs. It is suggested that you follow along with this guide for your first skill, and customize after getting this example to work. For this example, my publicly shared ThingSpeak Channel is used for the dynamic data source

For the first tab (Skill information), enter:

Name: ESP8266

Invocation Name: e s p eighty two sixty six dash twelve

Then click “Save” at the bottom.

For the Interaction Model tab, enter:

Intent Schema (copy/paste):
