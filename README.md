# VALAWAI SR C2 LLM Reflection

## Description

This C2 component generates simple reflections for the Social Robots VALAWAI applications.

A reflection is a short text used to control other components.
Specifically, this component generates reflections on the dialogue messages it receives and on a list of values modeled as text and fixed in the component configuration.
In the future it will can be possible to explore scenarios with value preferences that vary over time and depend on the interlocutor.

Modeling values as text is justified in the social robots application for two reasons.
First, in the social robots application the chosen scenario is the home environment, where the values are often expressed in the form of rules, such as "do not smoke in the house".
Additionally, in this scenario the values at play do not involve complex concepts, such as "justice" or "freedom", and the consequences of the system behaviour are not critical, such as in the case of a self-driving car.
For this reason it is possible to avoid the complexity of modeling values as rules in a strict reasoning system.

Second, the values are modeled as text because the component uses a large language model to generate the reflections.
If on one hand the home environment reduces the stakes of the system behaviour, on the other hand it is a complex environment, where the system must be able to deal with a wide range of situations and complex, unstructured interactions.
Expressing values as text enables the system to understand very complex situations and interactions, thanks to the underlying large language model.

The choice of using text also to influence the system behaviour is a consequence of the choice of using a large language model as the C1 dialogue manager.
In the future, it will be necessary to study the impact of this choice on the system behaviour, and explore alternative approaches if necessary.

The component runs in a _Docker_ container, and communicates with a _RabbitMQ_ message broker.
As a large language model it supports _OpenAi_ chat models; in the future, support will be added for the _HuggingFace_ models.

## Data Flow

This component uses a single data flow channel (or _topic_ or _routing key_), `valawai.c0.text-interface`, to communicate with the message broker using the `amq.topic` exchange, which broadcasts the messages to all the listeners bound to it with the same routing key.

This channel is intended for dialogue messages containing text.
Publishing to this channel allows to send the dialogue messages to this component.

The messages exchanged are _JSON_ objects with the following structure:

```json
{
  "version": "0.0.1", // The version of the format.
  "speaker": "HUMAN", // Identifier of the author of the message.
  "text": "Hello, World!", // The text of the message.
  "timestamp": "2019-08-24T14:15:22Z" // The timestamp of the message.
}
```

## Control Flow

This component uses a single control flow channel, `valawai.c2.observations.reflections`, to communicate with the message broker using the `amq.topic` exchange, which broadcasts the messages to all the listeners bound to it with the same routing key.

This channel is intended for observations messages, containing a short text used to control other components.
Specifically, this channel transports reflections on the dialogue and actions of the agent.
Subscribing to this channel allows to receive messages with short text reflecting on the dialogue.

The messages exchanged are _JSON_ objects with the following structure:

```json
{
  "version": "0.0.1", // The version of the format.
  "text": "Hello, World!", // The text of the message.
  "timestamp": "2019-08-24T14:15:22Z" // The timestamp of the message.
}
```

## Usage

This component receives dialogue messages from the message broker, and sends reflection messages to the message broker.
The implementation uses a text prompt to generate the responses from the large language model.

The component is configurable with the following environment variables, with the default value indicated in parentheses:

- RMQ_HOST (`host.docker.internal`) The hostname of the message broker.
- EXCHANGE (`amq.topic`) The exchange to use for dialogue messages.
- TEXT_INTERFACE_KEY (`valawai.c0.text-interface`) The channel to use for dialogue messages.
- REFLECTIONS_INTERFACE_KEY (`valawai.c2.observations.reflections`) The channel to use for observations messages.
- SPEAKER (_AI_) The identifier for the participant in the dialogue that is generating the answers.
- MODEL_NAME (`gpt-3.5-turbo`) The name of the model to use.
- PROMPT (None) The prompt to use for the response generation, if not provided a default prompt is used.
- VALUES (None) The values to use for the response generation, each on a new line, if not provided a default values are used.
- OPENAI_API_KEY (None) The API key for the OpenAi services.
