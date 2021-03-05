
# Build an Alexa Skill with Python and AWS lambda — a serverless love story

“white and black Amazon Echo dot” by Rahul Chakraborty on Unsplash

Its been almost an year since Amazon has started selling its device [Echo](https://en.wikipedia.org/wiki/Amazon_Echo) in India.
Its a brand of smart speakers integrated with an AI-based personal assistance which responds with name Alexa. Since its release, there have been thousands of skills pushed almost every day on [Alexa Indian skill store](https://www.amazon.in/b/?ie=UTF8&node=11928183031&ref=alexa_skills_exact_1) ranging from food ordering to movie reviews by developers across the world.

## What is an Alexa Skill and how does it work?

Skill is to Alexa what app is for Android or IOS. Alexa provides skills, which allows users to interact with the device. These skills can answer questions, play quizzes, share news updates or play music.

Every skill has two important components: A **skill interface** which you refer to as frontend of your skill and **skill service **which you can refer to as the backend of your skill.

## **JSON JSON Everywhere: The Skill Interface Part**

The first step to start receiving requests from the user. You need to start by configuring the model JSON which you can find on the left side with an option JSON Editor in your Alexa developer console.

Here is a sample JSON format of space/planet fact skills that tells you random facts about space and planets whenever you invoke it

    {
      "interactionModel": {
        "languageModel": {
          "invocationName": "space facts",
          "intents": [
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
              "name": "GetNewFactIntent",
              "samples": [
                "a space fact",
                "tell me a space fact",
                "give me a space fact",
                "tell me a space trivia",
                "give me a space trivia",
              ],
              "slots": []
            },
            {
              "name": "GetPlanetFactIntent",
              "samples": [
                "a space fact",
                "tell me some planets fact",
                "give me some planets fact",
                "tell me some planets trivia",
                "give me some planets trivia",
               ],
              "slots": []
            }      
            ]
        }
      }
    }

The JSON object will be represented by Python dictionaries where every element will have two parts: Name and samples. The name will be a unique identifier for different intents while samples will have different ways of how you want to invoke the intent. For eg GetNewFactIntent will invoke space facts while GetPlanetFactIntent will invoke planets facts.

## **Skill Service & **AWS Lambda: The future of serverless computing

The next part of building Alexa skill is to connect your skill interface with a backend service using AWS lambda. [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) is a compute service that lets you run code without managing any server. AWS Lambda executes your code only when needed and scales automatically, from a few requests per day to thousands per second. All you need to do is supply your code in one of the languages that AWS Lambda supports (currently Node.js, Java, C#, Go and Python).

Lambda functions starts with lambda_handler function. Think of it like a main().

    def lambda_handler(event, context):
                return "Hello World"

When you connect your Alexa skill interface with the endpoint, the skill interface starts invoking your lambda_handler function with every request.

The next step is to identify different intents that you created in your skill interface. Let's see an example how we can log different information based on the basis of different intent by looking at their names.

    def lambda_handler(event, context):
        if event['request']['type'] == "LaunchRequest":
                logger.info('LaunchRequest Intent Invoked')
        if event['request']['intent']['name'] == "GetSpaceIntent":
                logger.info('Space Intent Invoked')
        elif event['request']['intent']['name'] == "GetPlanetIntent":
                 logger.info('Planet Intent Invoked')

The last thing is to construct the response which Alexa Skill Interface can understand. So let's quickly construct that response for our skill interface

    def build_response(message, session_attributes={}):
        response = {}
        response['version'] = '1.0'
        response['sessionAttributes'] = session_attributes
        response['response'] = {'outputSpeech':message}
        return response
        
    def build_PlainSpeech(body):
        speech = {}
        speech['type'] = 'PlainText'
        speech['text'] = body
        return speech

    def lambda_handler(event, context):
        if event['request']['type'] == "LaunchRequest":
                message = build_PlainSpeech("Welcome to Space skill")
                return build_response(message)
        if event['request']['intent']['name'] == "GetSpaceIntent":
                message = build_PlainSpeech("Elon Musk loves SpaceX!")
                return build_response(message)
        elif event['request']['intent']['name'] == "GetPlanetIntent":
                message = build_PlainSpeech("Elon Musk loves Mars!")
                return build_response(message)

## Conclusion

I hope you found the blog useful. Although this was not a complete guide to making Alexa Skill with Python, it should get you started.

Message me on [Twitter](http://twitter.com/nishant483) or do leave a comment if you have any suggestion/request. Go out and build awesome Alexa Skills! If you publish a skill, please share it across. I would love to try it out.

***Learnt something useful from the article? Kindly show your appreciation by giving a clap and sharing it with your friends. ***:)
