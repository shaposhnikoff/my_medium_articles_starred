
# Simplified Header Based Routing with Istio for HTTP & GRPC Traffic

For different types of deployment strategies(like: blue-green, canary, A/B Testing), a proper management of traffic is required which could handle Autoscaling for multiple version based on its own usage. Not only the api gateway but we might need to split the traffic when working with gRPC services. There comes Istio which lets makes us easy to control fine-grained traffic.

![Image credit: kruschecompany.com](https://cdn-images-1.medium.com/max/2560/1*CCmupPBL2caw1fgdzt6H4g.jpeg)*Image credit: kruschecompany.com*

Let’s consider a scenario, you have 3 services: Api gateway, auth and article. A new version of article service is to be released but obviously we don’t want to surprise every user base but gradually rollout the version. We can plan something like, the new version will be available to users of paid plan in first phase. Now, in traditional way, we might think of handling all of those if-else logics on code base which makes things dirty if not out of control. But in the age of service mesh like Istio and microservice container based deployments, we can easily rollout the new version. We can plan something like this:

When a user under paid plan login to the system, we can attach a specific header(new-article-version: true)which will then be passed to the article service. And in between, we can apply Istio routing logic to segregate the traffic so that the user request with header new-article-version: true will land on newer version and rest of the users doesn’t notice the latest release. After the feedback of paid plan users, error analysis, application performance, we can gradually release the version to all users.

![Istio Header Based Routing](https://cdn-images-1.medium.com/max/2000/1*uR13zJoZyWpiuMLW9b9jQw.png)*Istio Header Based Routing*

Here, we have two versions of deployment running: green-article and grey-article which is denoted by green-grey of the Istio *DestinationRule*.

The above configuration works whether the article service is http or gRPC based.

Now, let’s think of another way of releasing to non-paid users too. We still don’t want to give surprise to all those users but want canary deployment where in first shot we choose 25% of non-paid users to enjoy the new version. Again, based on the feedback and interactions, we will open it to all. Its simple with Istio routing logic:

![Istio canary(percentage) traffic split](https://cdn-images-1.medium.com/max/2000/1*6ZOY5v2mThr5GlNiH9xUJA.png)*Istio canary(percentage) traffic split*

Here, we still respect our previous logic of releasing new version(grey) to paid users. Also, we are releasing the new version to 25% of non-paid users.

Isn’t that awesome? We can do these types of traffic splitting by just few lines of yaml configurations of Istio which could have taken days for managing via application logic or code.

Have you tried or faced situations like this before? Or you have more robust solution? Feel free to share.

Get connected with me on T[witter](https://twitter.com/dwdRAJU) and L[inkedin](https://www.linkedin.com/in/dwdraju/) where I keep on sharing interesting things.
