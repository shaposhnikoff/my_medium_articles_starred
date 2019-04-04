
# How To Run Migrations on AWS ECS and Other One off Tasks

How To Run Migrations on AWS ECS and Other One off Tasks

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/Lv4wcM0eCrI" frameborder="0" allowfullscreen></iframe></center>

Once a developer came up to me and told me that he was ready to deploy, but the deploy required migrations be run. He asked me an innocent question, how should he run migrations. Embarrassingly, because we were still evolving our tooling for the new infrastructure, we did not have a process for this simple task. We came up with an ad-hoc process that was honestly pretty terrible. It went like this:

Deploy the app with only the migrations and no other code changes.

1. Ssh into the server and docker exec into the container.

1. Run the migrations.

1. Deploy the app again with the new code changes.

It was such a terrible process that I would notice developers creatively taking their own “shortcut” route to save time:

1. Deploy the app with both the code changes and migrations together.

1. Hustle and ssh into the server and quickly docker exec into the container.

1. Run the migrations.

The second flow would cause issues when the new code ran while the migration still hadn’t been run. Even after the migrations were ran, the app still sometimes continue to error. We would then frantically restart the app to make all well again. I can sympathize developers trying to take the “shortcut” route. Having to do extra manually work at the very end when you have spent all week busting your hump writing code just does not jive well.

In this post, I’ll go over how to run Rails Migrations on AWS ECS in a sane manner without the extra manual work.

## ECS Task vs Service

Let’s think about the ways we can run docker containers with AWS ECS. ECS can both run a single one-off task and a long running-service task. The one-off ECS single task is a perfect fit for running migrations. What we need to do is:

1. Build a docker image.

1. Build and register a task definition.

1. Execute a one-off ECS task.

## Enter UFO

UFO is a tool that helps you easily build and ship your docker images to AWS ECS. UFO also comes with a handy ufo task command that allows you to run a one-off task. The documentation for it is available on the official ufo [Run Single Task](http://ufoships.com/docs/single-task/) documentation. It effectively automates the process mentioned above. Usage is simple:

    ufo task hi-web --command bundle exec rake db:migrate

One of the things you can do is set this up on a continuous integration provider like CircleCI or Jenkins, so all the developer needs to do is commit his or her migration to a specific migrate branch, and the CI will run the ufo task command.

The ufo task allows you to quickly deploy and run any code you’d like without lots of manual efforts. No more excuses to cause yourself pain 😁 Hope you found this helpful.
> # Thanks for reading this far. If you found this post useful, I’d really appreciate it if you recommend this post (by clicking the clap button) so others can find it too! Also, follow me on [Medium](https://medium.com/@tongueroo).

![](https://cdn-images-1.medium.com/max/2000/1*forZa8mfCGWYU5PAz-fRQA.gif)
> # P.S. Be sure to join the [BoltOps newsletter to receive free DevOps tips and updates.](https://www.boltops.com/subscribe)
