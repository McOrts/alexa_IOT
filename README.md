![Alexa IOT](https://github.com/McOrts/alexa_IOT/blob/master/images/mallorca_beach_weather_skill_poster.PNG?raw=true)
# Alexa te informa de la temperatura y el sol de la Playa de Palma 

No podía esperar para poder usar los comandos de voz de mi Alexa Echo para leer, encender y apagar dispositivos. Y hace un año monté la [estación metereológia móvil basada en el arduino MRKFOX1200](https://github.com/McOrts/MKRFOX1200_mobile-weather-station). Ahora cualquier Alexa del mundo puede informar de la temperatura, presión aatmosférica y radiación ultravioleta en la Playa de Palma.

  <a href="http://www.youtube.com/watch?feature=player_embedded&v=WW9ZDhAB9yA
  " target="_blank"><img src="http://img.youtube.com/vi/WW9ZDhAB9yA/0.jpg"
  alt="Basic video" width="240" height="180" border="10" /><

## Alexa conectada al IOT

El concepto es conectar un perfil (Skill) a un interface con un dispositivo IOT basado en Arduino para poder interactuar con el mundo físico. Desde encender una bombilla, o leer un sensor de temperatura, hasta controlar un robot remotamente. Todo empieza definiendo un _Invocation Name_ al que Alexa atenderá cuando le digamos: Alexa! En mi caso es:

_**Alexa! ask weather station**_

## Arquitectura

La solución presentada aquí utiliza los servicios web de Amazon (AWS) para hacer toda la interfaz de voz con Alexa y el backend, Things Speak como repositorio para los datos de los sensores del Arduino y la red SigFox para subir los datos a Things Speak. De esta manera tanto el Alexa como el dispositivo IOT pueden estar situados en casi cualquier parte del mundo y mantener su interconectividad.

![Arquitectura alexa ESP8266](https://github.com/McOrts/alexa_IOT/blob/master/images/alexa-esp-ts_architecure.jpg?raw=true)

Al ver el diagrama quizás te hayas preguntado ¿Por qué no comunicarse directamente con el ESP8266 desde la función AWS Lambda? Porque implicaría saltarse varias reglas de seguridad exponiendo las credenciales del dispositivo para que AWS pudiera acceder. Amazon tiene su propia solución para esto, el AWS Greengrass, pero implica un coste y es más complejo. Things Speak actua de cortafuego y nos permite el acceso a los datos desde múltiples clientes además de Alexa.

## ¿Qué necesitamos?
Los componentes utilizados para este proyecto son estos:
* ![Amazon Alexa Echo Dot](https://github.com/McOrts/alexa_IOT/blob/master/images/echo_dot.png?raw=true) [Amazon Alexa Echo Dot](http://amzn.eu/d/8Blx0LD)
* ![Alexa Skills Kit](https://github.com/McOrts/alexa_IOT/blob/master/images/alexa_skill.jpg?raw=true) [Alexa Skills Kit](https://developer.amazon.com/alexa/console/ask)
* ![AWS Lambda](https://github.com/McOrts/alexa_IOT/blob/master/images/aws_lambda.jpg?raw=true) [AWS Lambda](http://aws.amazon.com)
* ![ThingSpeak API](https://github.com/McOrts/alexa_IOT/blob/master/images/ThingSpeak.jpg?raw=true) [API de ThingSpeak](https://thingspeak.com)

## Implementación
Este proyecto tiene dos partes: la de Amazon y la de Arduino+ThingSpeak. Esta última está explicada en el proyecto [estación metereológia móvil basada en el arduino MRKFOX1200](https://github.com/McOrts/MKRFOX1200_mobile-weather-station) del que parte esta idea. Para la parte de Amazon recomiento este [repositorio oficial de Alexa](https://github.com/alexa/skill-sample-python-fact/tree/master/instructions) muy útil tanto para novatos como para iniciados. Contiene una documentación que te guiará paso por paso en la producción de un skill basado en código Python y que se resume en estos apartados:

![AWS_skill_workflow](https://github.com/McOrts/alexa_IOT/blob/master/images/AWS_skill_workflow.PNG?raw=true)

### 1 Montando el diálogo (Voice User Interface)
Hay que diseñar el dialogo más en la parte de la respuesta, que estará toda descrita en el programa Python que en las preguntas. Amazon ha desarrollado una configuración de metadatos que simplifica extremadamente esta tarea. No tenemos que saber nada de reconocimiento de lenguaje natural. Solo hay definir dos elementos: el **Invocation name** que es la clave de llamada a nuestro skill y los **Intents** que son ejemplos de complementos directos de la sintaxis de la frase. No es necesario poner todas las posibilidades. La IA de Alexa sabrá interpretar las variantes de las preguntas del usuario.

En mi caso, pretendía algo simple como esto:

**Comando vocal del usuario:**
_“Alexa, ask weather station for measures“_

**Respuesta de Alexa:**
_“The temperature is 13 degrees the index of ultraviolet radiation is -1.2 and the atmospheric pressure is 1016“_

Todo esto se configura en el _Interaction Model_ que construye un modelo que el frontal de Alexa ejecuta para interactuar con el usuario. A efectos prácticos esto queda resumido en una estructura JSON que mi proyectos es esta:

```
{
    "interactionModel": {
        "languageModel": {
            "invocationName": "weather station",
            "intents": [
                {
                    "name": "AMAZON.FallbackIntent",
                    "samples": []
                },
                {
                    "name": "AMAZON.CancelIntent",
                    "samples": []
                },
                {
                    "name": "AMAZON.HelpIntent",
                    "samples": []
                },
                {
                    "name": "AMAZON.StopIntent",
                    "samples": []
                },
                {
                    "name": "AMAZON.NavigateHomeIntent",
                    "samples": []
                },
                {
                    "name": "GetEspInfoIntent",
                    "slots": [],
                    "samples": [
                        "for measures",
                        "data",
                        "for information"
                    ]
                }
            ],
            "types": []
        }
    }
}
```

Por otra parte, este frontal necesita y saber a dónde tiene que llamar. Esto se informa en el apartado *Endpoint* donde hay decir si se llamará una aplicación _serverless_ de AWS (Lambda) o a un servicio web externo. En mi caso es un Lambda y aquí tendremos que poner si identificador que viene informado en AWS como ARN.

### 2 Montando el backend (Lambda Function)
En primer lugar necesitaremos una cuenta de usuario diferente. No puede ser la misma que hemos utilizado para el skill. 
Aquí tenemos que implementar la lógica de la llamada a ThingSpeak para recuperar los datos de la estación y elaborar las respuestas que Alexa va a reproducir.

Yo me he basado en el ejemplo del [repositorio oficial de Alexa](https://github.com/alexa/skill-sample-python-fact/tree/master/instructions) añadiendo la libreria _urllib_ para poder hacer la llamada a la API de ThingSpeak. Es importante saber que hay que compilar las dependencias en un servidor que tenga el [ASK SDK for Python](https://github.com/alexa/alexa-skills-kit-sdk-for-python). Yo he utilizado una instancia S3 básica para hacer esta compilación y generar todos los archivos que Lambda va a necesitar y utilizando el servicio SCP para descargarme el .zip. Está todo muy bien explicado en el punto 7 del documento: https://github.com/alexa/skill-sample-python-fact/blob/master/instructions/2-lambda-function.md

Y aquí está el código Python:

```
# -*- coding: utf-8 -*-
"""Simple fact sample app."""

import random
import logging
import urllib

from ask_sdk_core.skill_builder import SkillBuilder
from ask_sdk_core.dispatch_components import (
    AbstractRequestHandler, AbstractExceptionHandler,
    AbstractRequestInterceptor, AbstractResponseInterceptor)
from ask_sdk_core.utils import is_request_type, is_intent_name
from ask_sdk_core.handler_input import HandlerInput

from ask_sdk_model.ui import SimpleCard
from ask_sdk_model import Response


# =========================================================================================================================================
# TODO: The items below this comment need your attention.
# =========================================================================================================================================
# Change these elements to point to your data

# Thingspeak configuration
channel = 365024
link_pressure = "https://api.thingspeak.com/channels/" + str(channel) + "/fields/3/last"
link_temperature = "https://api.thingspeak.com/channels/" + str(channel) + "/fields/2/last"
link_uv = "https://api.thingspeak.com/channels/" + str(channel) + "/fields/4/last"

# Alexa params
SKILL_NAME = "Weather Station"
HELP_MESSAGE = "You can say tell me weather information, or, you can say exit... What can I help you with?"
HELP_REPROMPT = "What can I help you with?"
STOP_MESSAGE = "Goodbye!"
FALLBACK_MESSAGE = "The weather station can't help you with that.  It can help you know weather information in Playa De Palma if you say ask weather station data. What can I help you with?"
FALLBACK_REPROMPT = 'What can I help you with?'
EXCEPTION_MESSAGE = "Sorry. I cannot help you with that."

# =========================================================================================================================================
# Editing anything below this line might break your skill.
# =========================================================================================================================================

sb = SkillBuilder()
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)


# Built-in Intent Handlers
class GetNewFactHandler(AbstractRequestHandler):
    """Handler for Skill Launch and GetNewFact Intent."""
    def can_handle(self, handler_input):
        # type: (HandlerInput) -> bool
        return (is_request_type("LaunchRequest")(handler_input) or
                is_intent_name("GetEspInfoIntent")(handler_input))

    def handle(self, handler_input):
        # type: (HandlerInput) -> Response
        logger.info("In GetNewFactHandler")

        f = urllib.urlopen(link_temperature) # Get data 
        weather_temperature = str(f.read())
        weather_data = weather_temperature
        speech = "the temperature is " + weather_temperature + " degrees"

        f = urllib.urlopen(link_uv) # Get data 
        weather_uv = str(round(float(f.read()),1))
        weather_data = weather_data + ";" + weather_uv
        speech = speech + " the index of ultraviolet radiation is " + weather_uv

        f = urllib.urlopen(link_pressure) # Get data 
        weather_pressure = str(f.read())
        weather_data = weather_data + ";" + weather_pressure
        speech = speech + " and the atmospheric pressure is " + weather_pressure


        handler_input.response_builder.speak(speech).set_card(
            SimpleCard(SKILL_NAME, weather_data))
        return handler_input.response_builder.response


class HelpIntentHandler(AbstractRequestHandler):
    """Handler for Help Intent."""
    def can_handle(self, handler_input):
        # type: (HandlerInput) -> bool
        return is_intent_name("AMAZON.HelpIntent")(handler_input)

    def handle(self, handler_input):
        # type: (HandlerInput) -> Response
        logger.info("In HelpIntentHandler")

        handler_input.response_builder.speak(HELP_MESSAGE).ask(
            HELP_REPROMPT).set_card(SimpleCard(
                SKILL_NAME, HELP_MESSAGE))
        return handler_input.response_builder.response


class CancelOrStopIntentHandler(AbstractRequestHandler):
    """Single handler for Cancel and Stop Intent."""
    def can_handle(self, handler_input):
        # type: (HandlerInput) -> bool
        return (is_intent_name("AMAZON.CancelIntent")(handler_input) or
                is_intent_name("AMAZON.StopIntent")(handler_input))

    def handle(self, handler_input):
        # type: (HandlerInput) -> Response
        logger.info("In CancelOrStopIntentHandler")

        handler_input.response_builder.speak(STOP_MESSAGE)
        return handler_input.response_builder.response


class FallbackIntentHandler(AbstractRequestHandler):
    """Handler for Fallback Intent.
    AMAZON.FallbackIntent is only available in en-US locale.
    This handler will not be triggered except in that locale,
    so it is safe to deploy on any locale.
    """
    def can_handle(self, handler_input):
        # type: (HandlerInput) -> bool
        return is_intent_name("AMAZON.FallbackIntent")(handler_input)

    def handle(self, handler_input):
        # type: (HandlerInput) -> Response
        logger.info("In FallbackIntentHandler")

        handler_input.response_builder.speak(FALLBACK_MESSAGE).ask(
            FALLBACK_REPROMPT)
        return handler_input.response_builder.response


class SessionEndedRequestHandler(AbstractRequestHandler):
    """Handler for Session End."""
    def can_handle(self, handler_input):
        # type: (HandlerInput) -> bool
        return is_request_type("SessionEndedRequest")(handler_input)

    def handle(self, handler_input):
        # type: (HandlerInput) -> Response
        logger.info("In SessionEndedRequestHandler")

        logger.info("Session ended reason: {}".format(
            handler_input.request_envelope.request.reason))
        return handler_input.response_builder.response


# Exception Handler
class CatchAllExceptionHandler(AbstractExceptionHandler):
    """Catch all exception handler, log exception and
    respond with custom message.
    """
    def can_handle(self, handler_input, exception):
        # type: (HandlerInput, Exception) -> bool
        return True

    def handle(self, handler_input, exception):
        # type: (HandlerInput, Exception) -> Response
        logger.info("In CatchAllExceptionHandler")
        logger.error(exception, exc_info=True)

        handler_input.response_builder.speak(EXCEPTION_MESSAGE).ask(
            HELP_REPROMPT)

        return handler_input.response_builder.response


# Request and Response loggers
class RequestLogger(AbstractRequestInterceptor):
    """Log the alexa requests."""
    def process(self, handler_input):
        # type: (HandlerInput) -> None
        logger.debug("Alexa Request: {}".format(
            handler_input.request_envelope.request))


class ResponseLogger(AbstractResponseInterceptor):
    """Log the alexa responses."""
    def process(self, handler_input, response):
        # type: (HandlerInput, Response) -> None
        logger.debug("Alexa Response: {}".format(response))


# Register intent handlers
sb.add_request_handler(GetNewFactHandler())
sb.add_request_handler(HelpIntentHandler())
sb.add_request_handler(CancelOrStopIntentHandler())
sb.add_request_handler(FallbackIntentHandler())
sb.add_request_handler(SessionEndedRequestHandler())

# Register exception handlers
sb.add_exception_handler(CatchAllExceptionHandler())

# TODO: Uncomment the following lines of code for request, response logs.
# sb.add_global_request_interceptor(RequestLogger())
# sb.add_global_response_interceptor(ResponseLogger())

# Handler name that is used on AWS lambda
lambda_handler = sb.lambda_handler()
```
### 3. Conectar AWS con ALEXA (Connect VUI to Code)
Como he comentado antes existe un código identificador llamado ARN que debemos relacionar copiandolo de la parte superior consola de AWS al apartado Endpoint del consola de Skills de ALEXA.

### 4. testing
Se pueden hacer pruebas desde los dos entornos. El de AWS nos probará la función Lambda en el modo _Fact_ que cuando se produce ninguna excepción y que además podemos debugar usando los logs de CloudWatch.
![Test de Lambda](https://github.com/McOrts/alexa_IOT/blob/master/images/test_lambda.PNG?raw=true)

Y por otra parte la consola de desarrollo de Alexa nos deja simular el comando de voz completo, incluso usando el micrófono de nuestro ordenador:

![Test de skill](https://github.com/McOrts/alexa_IOT/blob/master/images/test_skill.PNG?raw=true)

### 6. Publicación
Esto es otra aventura que explicaré más adelante y que ya he empezado:
![Test de skill](https://github.com/McOrts/alexa_IOT/blob/master/images/mallorca_beach_weather_skill_entry.PNG?raw=true)

# Conclusiones
Hacer este tipo de desarrollos no es trivial. Además el uso de recursos de AWS tiene un coste que puede llevar a morir de éxito a tu skill si no le has hecho un buen plan de monetización. Amazon tiene presente este problema y ofrece [créditos promocionales a desarrolladores](https://developer.amazon.com/es/alexa-skills-kit/alexa-aws-credits) que desplieguen en esta tecnología y que la cuenta gratuita (AWS Free Tier) no sea suficiente con el millón de peticiones al mes que incluye sin coste.


