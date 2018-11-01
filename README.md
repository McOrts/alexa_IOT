# Alexa te informa de la temperatura y el sol de la Playa de Palma 

No podía esperar para poder usar los comandos de voz de mi Alexa Echo para leer, encender y apagar dispositivos. Y hace un año monté la [estación metereológia móvil basada en el arduino MRKFOX1200](https://github.com/McOrts/MKRFOX1200_mobile-weather-station). Ahora cualquier Alexa del mundo puede informar de la temperatura, presión atmostférica y radiación untravioleta en la Playa de Palma.



Uttering Alexa voice commands to turn your ESP8266 connected device on or off is cool. There is plenty of information readily available explaining how to make that happen.
Uttering Alexa voice commands to turn your ESP8266 connected device on or off is cool. There is plenty of information readily available explaining how to make that happen.

But what a custom command?

Like having Alexa tell you something unique. Using data gathered by reading your specific IoT device sensors?

Such as ESP8266 measured values that change over time.

For example, temperature or other weather sensors.

Having the ability to interact verbally with your IoT device in this manner opens up a wide range of possibilities. Yet, as I discovered, information on this topic was more challenging to find with a google search.

So here is how I did it.

And so can you.

The Problem
What is needed is a method of converting your verbal request into a command that can be sent to your ESP8266 (or other IoT device) to get the requested information, and to return the information to Alexa in a format that she can read back to you.

The solution presented here uses the Amazon Web Services (AWS) to do all the voice interfacing with Alexa, Things Speak as a repository for the ESP8266 sensor data, and a VPS CRON script to move the sensor data from the ESP8266 to Things Speak.

You might wonder why a VPS and Thing Speak are used. Why not communicate directly with the ESP8266 from the AWS Lambda Function?

Truth is, you can cut out these intermediate steps. But for security, I have added this layer to hide my IoT credentials within my heavily fortified Virtual Private Server (VPS). But do not be concerned, as you will see, this does not make the system overly complicated.

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
