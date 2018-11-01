# Alexa te informa de la temperatura y el sol de la Playa de Palma 

No podía esperar para poder usar los comandos de voz de mi Alexa Echo para leer, encender y apagar dispositivos. Y hace un año monté la [estación metereológia móvil basada en el arduino MRKFOX1200](https://github.com/McOrts/MKRFOX1200_mobile-weather-station). Ahora cualquier Alexa del mundo puede informar de la temperatura, presión atmostférica y radiación untravioleta en la Playa de Palma.

video

## Alexa conectada al IOT

El concepto es conectar un perfil (Skill) a un interface con un dispositivo IOT basado en el ESP8266 para poder interactuar con el mundo físico. Desde encender una bombilla, o leer un sensor de temperatura, hasta controlar un robot remotamente. Y tendremos que empezar por definir un _Invocation Name_ al que Alexa atienda cuando le digamos: Alexa! En mi caso:

Alexa! ask weather station

## Arquitectura

La solución presentada aquí utiliza los servicios web de Amazon (AWS) para hacer toda la interfaz de voz con Alexa y el backend, Things Speak como repositorio para los datos de los sensores del ESP8266 y la red SigFox para subir los datos a Things Speak. De esta manera tanto el Alexa como el dispositivo IOT pueden estar situados en casi cualquier parte del mundo y mantener su interconectividad.

diagrama

Al ver el diagrama quizás te hayas preguntado ¿Por qué no comunicarse directamente con el ESP8266 desde la función AWS Lambda? Porque implicaría saltarse varias reglas de seguridad exponiendo las credenciales del dispositivo para que AWS pudiera acceder. Amazon tiene su propia solución para esto, el AWS Greengrass, pero implica un coste y es más complejo. Things Speak actua de cortafuego y nos permite el acceso a los datos desde multiples clientes además de Alexa.


System Diagram


Test Scenario
We start with a voice command: “Alexa, ask ESP8266-12 for the indoor temperature”

Alexa responds with: “The indoor temperature is 73.2 degrees”

So what just happened?

Since we are dealing with the Amazon Echo device, Amazon lingo is needed (bold).

The voice command was received by the Amazon Echo device and recognized as an Alexa skill.

Invocation of the skill triggered the Lambda function which, in-turn, sent a request to ThingSpeak for the current ESP8266 sensor readings,

The readings returned were passed back to the Alexa skill which recites the values of the temperature sensors.

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
