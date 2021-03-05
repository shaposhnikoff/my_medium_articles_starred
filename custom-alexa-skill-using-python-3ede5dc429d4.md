
# Custom Alexa Skill using Python

Amazon’s Alexa is the voice-activated, interactive AI bot, or personal assistant, that lets people speak with their Amazon Echo, Echo Dot and other Amazon smart home devices. Like Siri and Cortana, Alexa is designed to respond to a number of different commands and even converse with users. Alexa comes with more than a few capabilities: playing music, pulling up the weather or even reading news. But Alexa Skills are apps that give Alexa even more abilities, letting her speak to more devices.

Alexa is Amazon’s cloud-based voice service. It is available on tens of millions of devices from Amazon and third-party device manufacturers. With Alexa, We can build natural voice experiences that offer customers a more intuitive way to interact with the technology they use every day.

Alexa works around below points :

1)Invocation Name: Use a phrase supported by the Alexa service in combination with the invocation name for a custom skill to request information or load custom skill. For example Alexa play music, Alexa any custom skill.

2) Intent : An intent represent actions that users can do with skill. These intents represent the core functionality of skill . It is a dialog or a conversation with customers in which Alexa asks questions and the user responds with the answers

3) Slots : slots is an optional parameters that can be used to pass dynamic query. For e.g. below intent sample has slot “query”

find Quiz for {query}”, “find {query} Quiz”.

4)Endpoint : Endpoint is a API or Lambda address where Alexa skill send and receive response.

To create Custom Alexa skill we need to create login on [**https://developer.amazon.com](https://developer.amazon.com/) **and then we can configure required intent , slot and endpoints using wizard or define using Json.

High level design

![](https://cdn-images-1.medium.com/max/2000/1*wP72Gdx3wcniloazLT1-3Q.png)

To Get Profile information of user, We can use Alexa account linking feature [https://developer.amazon.com/docs/account-linking/understand-account-.linking.html](https://developer.amazon.com/docs/account-linking/understand-account-linking.html) Below is the configuration of account linking

![](https://cdn-images-1.medium.com/max/2694/1*nCTNcDfdvVWGl14M48rAiw.png)

Python code to get Email and Name from profile

if not (req_envelope.context.system.user.permissions and req_envelope.context.system.user.permissions.consent_token):
 response_builder.speak(“Please enable profile permissions in the Amazon Alexa app.”)
 response_builder.set_card(AskForPermissionsConsentCard(permissions=[“alexa::profile:email:read”]))
 return response_builder.response 
 else:
 logger.info(“In LaunchRequest user found”)
 try: 
 service_client_fact = handler_input.service_client_factory 
 device_email_client = service_client_fact.get_ups_service() 
 email = device_email_client.get_profile_email()
 handler_input.attributes_manager.session_attributes[‘profileName’] = device_email_client.get_profile_name()
 handler_input.attributes_manager.session_attributes[‘email’] = email 
 logger.info(“email found “ + str (email)) 
 except Exception as e:
 logger.info(str(e))
 raise e

Typical design of Alexa Quiz App

![](https://cdn-images-1.medium.com/max/2000/1*e3_uxS5WIiLqAuoXa9OIFw.png)
