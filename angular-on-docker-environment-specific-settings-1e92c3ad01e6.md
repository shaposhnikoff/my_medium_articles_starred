
# Angular on Docker — environment specific settings

Angular on Docker — environment specific settings

Angular applications often have settings which vary depending on which environment they are running in. For example, when running it in production, an application may need to use a different URL to connect to a backend api, compared with a staging environment, or locally on a developer’s machine.

One solution is to put the settings in environment.ts file, and include an additional environment.{env-name}.ts files for each environment we need. This means we need to specify the environment when we build our application, which isn’t great when we want to deploy an image.

When the Angular application is deployed as a docker image, we really want the same image be used, regardless of which environment we’re deploying to, so a different solution is needed.

Firstly, we need to put the settings we need for local development into a json file. I put this file in the assets folder, which gets copied to the dist folder when the project is built (assuming you’re using angular cli).

<iframe src="https://medium.com/media/17e3758b6eb5a55ed23a745e5e9e56f4" frameborder=0></iframe>

Then we create a class to represent the settings in our Angular application:

<iframe src="https://medium.com/media/5132695556815f7ad55fa44a519ff70a" frameborder=0></iframe>

a service which can be injected into any class which needs access to the settings:

<iframe src="https://medium.com/media/946f0f40c4a3ffeb789d6015ecbcab88" frameborder=0></iframe>

and another service to request the settings on application startup:

<iframe src="https://medium.com/media/4d19cb889e6665577eadd930a012743f" frameborder=0></iframe>

This service needs to be registered as an APP_INITIALIZER in the app.module.ts:

<iframe src="https://medium.com/media/04e8df73fd7f45edaffe39f2ecd27c02" frameborder=0></iframe>

With this in place, the application should load the settings provided in the settings.json file on startup.

To vary the settings based on environment, I add an additional file in the assets folder.

<iframe src="https://medium.com/media/297e8bbb1744ae4a4ceac0571cdb3f65" frameborder=0></iframe>

This has the same structure as the settings.json file, but specifies the name of an environment variable, instead of the setting value.

Then I modify my Dockerfile, so that when the container starts up, it replaces the environment variable names in the template file, with the values of those variables, and overwrites the settings.json file with the result, before starting the webserver.

<iframe src="https://medium.com/media/ab417cc8a28af51f8e812d52b4524cb1" frameborder=0></iframe>

In this case I’m using the standard nginx image — if you’re using something different you’ll need to make the appropriate changes.

I use envsubst to perform the variable substitution, and then I startup nginx.

Now when I start up the container, I can pass in my settings as environment variables.

So now a single image can be deployed to any number of environments, and the settings varied accordingly.

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Join our community Slack and read our weekly Faun topics ⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
