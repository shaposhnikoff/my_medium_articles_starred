
# Build Alexa Custom Skill — Using AWS Lambda, Python & PyCharm



**Introduction: **This post provides a very basic and beginner friendly overview on how to build an Alexa Custom Skill using Python and deploy it to AWS.

In this post we will learn to build our very own custom Alexa Skill called “Super Heroes”.

**Prerequisites: **Before we get started, it would be great if you have basic knowledge of Python and AWS Lambda. However, if you just want to deploy a working POC and learn things on the fly, this course is for you.

[PyCharm](https://www.jetbrains.com/pycharm/) IDE from JetBrain.

## Step 1 — Setup Super Heroes skill in Alexa Console

* Create/Sign-in to your [developer.amazon](https://developer.amazon.com) account & click on Amazon Alexa on the dashboard.

* Open developer console by selecting skill builder>developer console:
[https://developer.amazon.com/alexa/console/ask](https://developer.amazon.com/alexa/console/ask) to create & manage your custom skills.
Proceed by naming your skill eg: “Super Heroes” and click on “Create Skill”.

![](https://cdn-images-1.medium.com/max/6716/1*75Y6Ddh2Z3CgUhjenulxWw.png)

* Enter the skill name, select your default language and select “Custom” for creating your custom interaction model.

* Under the ‘Build tab’, select invocation. 
Here you can setup a skill invocation phrase that you would use to invoke your custom skill.

![](https://cdn-images-1.medium.com/max/6720/1*z2_pnb07OUrvcf5NFvGqyQ.png)

* In Intents create a new ‘HeroInfo’ intent with ‘hero’ slot value.

![](https://cdn-images-1.medium.com/max/2560/1*m4M4u3LWn-EWbxgODRJ6Xw.jpeg)

* Enter values for hero slot.

![](https://cdn-images-1.medium.com/max/2560/1*Y0qAYQ7sYXsLMFUGqXigNw.jpeg)

* Click on build model button, it will take few seconds to build.

* To know more about utterances and slots visit [here](https://developer.amazon.com/en-US/docs/alexa/custom-skills/best-practices-for-sample-utterances-and-custom-slot-type-values.html).

## Step 2 — Setting up Environment

### Install Python

[https://www.python.org/downloads/](https://www.python.org/downloads/)

### Installing the AWS CLI

[https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

### Installing AWS-SAM (Serverless Application Model)

[https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)

### Setup AWS Credentials

[https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-getting-started-set-up-credentials.html](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-getting-started-set-up-credentials.html)

## Step 3— Lets get Coding

Open PyCharm and create a new project and select **AWS Serverless Application**

![](https://cdn-images-1.medium.com/max/3556/1*vjVTEP5Wi6Yq-QhjY6xKcg.png)

A default template will be created using SAM.

Open **app.py **and add following code:

    **import **logging
    **import **prompts
    
    **from **ask_sdk_core.skill_builder **import **SkillBuilder
    **from **ask_sdk_core.dispatch_components **import **(AbstractRequestHandler, AbstractExceptionHandler,
                                                  AbstractRequestInterceptor, AbstractResponseInterceptor)
    **from **ask_sdk_core.utils **import **is_request_type, is_intent_name
    **from **ask_sdk_core.handler_input **import **HandlerInput
    **from **ask_sdk_model **import **Response
    
    sb = SkillBuilder()
    logger = logging.getLogger(__name__)
    logger.setLevel(logging.DEBUG)
    
    
    *# Built-in Intent Handlers
    ***class **LaunchRequestHandler(AbstractRequestHandler):
        *"""Handler for Launch Intent."""
    
        ***def **can_handle(self, handler_input):
            *# type: (HandlerInput) -> bool
            ***return **is_request_type(**"LaunchRequest"**)(handler_input)
    
        **def **handle(self, handler_input):
            *# type: (HandlerInput) -> Response
            *logger.info(**"In LaunchRequestHandler"**)
            **return **handler_input.response_builder.speak(prompts.WELCOME_MESSAGE).ask(prompts.WELCOME_MESSAGE).response
    
    
    **class **HelpIntentHandler(AbstractRequestHandler):
        *"""Handler for Help Intent."""
    
        ***def **can_handle(self, handler_input):
            *# type: (HandlerInput) -> bool
            ***return **is_intent_name(**"AMAZON.HelpIntent"**)(handler_input)
    
        **def **handle(self, handler_input):
            *# type: (HandlerInput) -> Response
            *logger.info(**"In HelpIntentHandler"**)
            **return **handler_input.response_builder.speak(prompts.HELP_MESSAGE).response
    
    
    **class **CancelOrStopIntentHandler(AbstractRequestHandler):
        *"""Single handler for Cancel and Stop Intent."""
    
        ***def **can_handle(self, handler_input):
            *# type: (HandlerInput) -> bool
            ***return **(is_intent_name(**"AMAZON.CancelIntent"**)(handler_input) **or
                    **is_intent_name(**"AMAZON.StopIntent"**)(handler_input))
    
        **def **handle(self, handler_input):
            *# type: (HandlerInput) -> Response
            *logger.info(**"In CancelOrStopIntentHandler"**)
            **return **handler_input.response_builder.speak(prompts.STOP_MESSAGE).response
    
    
    **class **FallbackIntentHandler(AbstractRequestHandler):
        *"""Handler for Fallback Intent.
        AMAZON.FallbackIntent is only available in en-US locale.
        This handler will not be triggered except in that locale,
        so it is safe to deploy on any locale.
        """
    
        ***def **can_handle(self, handler_input):
            *# type: (HandlerInput) -> bool
            ***return **is_intent_name(**"AMAZON.FallbackIntent"**)(handler_input)
    
        **def **handle(self, handler_input):
            *# type: (HandlerInput) -> Response
            *logger.info(**"In FallbackIntentHandler"**)
    
            speech = prompts.FALLBACK_MESSAGE
            reprompt = prompts.FALLBACK_REPROMPT
            handler_input.response_builder.speak(speech).ask(reprompt)
            **return **handler_input.response_builder.response
    
    
    **class **SessionEndedRequestHandler(AbstractRequestHandler):
        *"""Handler for Session End."""
    
        ***def **can_handle(self, handler_input):
            *# type: (HandlerInput) -> bool
            ***return **is_request_type(**"SessionEndedRequest"**)(handler_input)
    
        **def **handle(self, handler_input):
            *# type: (HandlerInput) -> Response
            *logger.info(**"In SessionEndedRequestHandler"**)
    
            logger.info(**"Session ended reason: {}"**.format(
                handler_input.request_envelope.request.reason))
            **return **handler_input.response_builder.response
    
    
    *# Exception Handler
    ***class **CatchAllExceptionHandler(AbstractExceptionHandler):
        *"""Catch all exception handler, log exception and
        respond with custom message.
        """
    
        ***def **can_handle(self, handler_input, exception):
            *# type: (HandlerInput, Exception) -> bool
            ***return True
    
        def **handle(self, handler_input, exception):
            *# type: (HandlerInput, Exception) -> Response
            *logger.info(**"In CatchAllExceptionHandler"**)
            logger.error(exception, exc_info=**True**)
    
            handler_input.response_builder.speak(prompts.ERROR_MESSAGE).ask(prompts.HELP_REPROMPT)
            **return **handler_input.response_builder.response
    
    
    *# Request and Response loggers
    ***class **RequestLogger(AbstractRequestInterceptor):
        *"""Log the alexa requests."""
    
        ***def **process(self, handler_input):
            *# type: (HandlerInput) -> None
            *logger.debug(**"Alexa Request: {}"**.format(
                handler_input.request_envelope.request))
    
    
    **class **ResponseLogger(AbstractResponseInterceptor):
        *"""Log the alexa responses."""
    
        ***def **process(self, handler_input, response):
            *# type: (HandlerInput, Response) -> None
            *logger.debug(**"Alexa Response: {}"**.format(response))
    
    
    *# Custom intent request handler
    ***class **HeroInfoRequest(AbstractRequestHandler):
        *"""Handler for hero info request"""
    
        ***def **can_handle(self, handler_input):
            *# type: (HandlerInput) -> bool
            ***return **is_intent_name(**"HeroInfo"**)(handler_input)
    
        **def **handle(self, handler_input):
            slots = handler_input.request_envelope.request.intent.slots
    
            hero_dict = slots[**'hero'**]
            hero_name = hero_dict.value
            hero_info = prompts.get_hero_info(hero_name)
    
            logger.debug(**"Hero information: {}"**.format(hero_info))
            **return **handler_input.response_builder.speak(hero_info).ask(prompts.FALLBACK_REPROMPT).response
    
    
    *# Register intent handlers
    *sb.add_request_handler(LaunchRequestHandler())
    sb.add_request_handler(HelpIntentHandler())
    sb.add_request_handler(CancelOrStopIntentHandler())
    sb.add_request_handler(FallbackIntentHandler())
    sb.add_request_handler(SessionEndedRequestHandler())
    
    *# Register custom intents handler
    *sb.add_request_handler(HeroInfoRequest())
    
    *# Register exception handlers
    *sb.add_exception_handler(CatchAllExceptionHandler())
    
    *# Register request and response interceptors
    *sb.add_global_request_interceptor(RequestLogger())
    sb.add_global_response_interceptor(ResponseLogger())
    
    *# Handler name that is used on AWS lambda
    *handler = sb.lambda_handler()

Create a new Python file called as prompts.py for speech responses. You can modify the responses as per your likes.

    *# Response messages
    *WELCOME_MESSAGE = **"Hi, welcome to Super Heroes, you can ask me about your favorite Super Heroes!"
    **HELP_MESSAGE = **"You can ask me about a super hero, or, you can say exit... What can I help you with?"
    **HELP_REPROMPT = **"What can I help you with?"
    **FALLBACK_MESSAGE = **"I can't help you with that. It can tell you about super heroes"**,
    FALLBACK_REPROMPT = **"What can I help you with"
    **ERROR_MESSAGE = **"Sorry, an error occurred."
    **STOP_MESSAGE = **"Ok Bye"
    
    
    ***# Provides hero details
    ***def **get_hero_info(hero_name):
        hero = hero_name.lower().replace(**'-'**, **''**).replace(**' '**, **''**)
        **if **hero == **'superman'**:
            **return 'Dude wears underwear on top of his pants'
        elif **hero == **'batman'**:
            **return 'Na na na na na na na na Batman! He is a freaking weirdo, eehhh!'
        elif **hero == **'spiderman'**:
            **return 'Spider Man isn\'t the only one who gets his hand sticky,. After using the web...'
        elif **hero == **'wonderwoman'**:
            **return 'I am Wonder Woman I wonder where I left my keys. I wonder where I left my purse. I wonder where my ' **\
                   **'money went '
        elif **hero == **'ironman'**:
            **return 'In fourth grade, Tony\'s, robotics teacher told him to build a little robot for the science fair, ' **\
                   **'he finished it in a few hours and called it Optimus Prime! '
        else**:
            **return 'No data found'**

Finally update your template.yaml file

    **AWSTemplateFormatVersion**: **'2010-09-09'
    Transform**: AWS::Serverless-2016-10-31
    **Description**: Alexa Super Heroes using python
    
    **Globals**:
      **Function**:
        **Timeout**: 3
    
    **Resources**:
      **SuperHeroesFunction**:
        **Type**: AWS::Serverless::Function
        **Properties**:
          **CodeUri**: function/
          **Handler**: app.handler
          **Runtime**: python3.7
          **Layers**:
            - arn:aws:lambda:us-east-1:173334852312:layer:ask-sdk-for-python-36:1
          **Events**:
            **Alexa**:
              **Type**: AlexaSkill
    
    **Outputs**:
      **SuperHeroesFunction**:
        **Description**: **"Super Heroes Lambda Function ARN"
        Value**: !GetAtt SuperHeroesFunction.Arn
      **SuperHeroesFunctionIamRole**:
        **Description**: **"Implicit IAM Role created for Super Heroes function"
        Value**: !GetAtt SuperHeroesFunctionRole.Arn

Select your region and AWS Credentials

![](https://cdn-images-1.medium.com/max/2548/1*42fk3OpyXTzSDPUDpPYQNg.jpeg)

Now right click on your project and select “Deploy Serverless Application”

![](https://cdn-images-1.medium.com/max/2000/1*obCneHiu9brqal1yDBG2Sg.jpeg)

Create a new Stack & S3 bucket to deploy your project

![](https://cdn-images-1.medium.com/max/2000/1*8jfMtOwxF8h4nf77cTTS7Q.jpeg)

Once deployment is completed, click on AWS-Explorer>Lambda> Right click on newly deployed Lambda function and copy its ARN.

![](https://cdn-images-1.medium.com/max/2000/1*Llxq9KVX8RkUSK70Ep9QBw.jpeg)

Now go back to Alexa Console> Super Heroes Skill and select Endpoints.
Enter copied ARN in Default Region and hit Save Endpoint

![](https://cdn-images-1.medium.com/max/2560/1*P5xujhSSBB_D7Q4u9hKkUQ.png)

## Step 4— Testing Super Heroes

In developer console click on Test and enable testing in development.

You should now be able to test your custom skill in the Simulator or an Echo device logged in with your developer account.

![](https://cdn-images-1.medium.com/max/2560/1*9cmlB6byDa31_mYUhfrDpg.png)

## Resources:

[https://github.com/cyberinsane/alexa-super-heroes](https://github.com/cyberinsane/alexa-super-heroes)
