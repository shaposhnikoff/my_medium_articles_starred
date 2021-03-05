
# How to setup your first github packages docker repository

On October 30, 2020 Docker Inc. announced docker pull rate limits will be implemented. This caused CI builds to be rate limited and fail. One way to solve being rate limited by Docker Hub is to set up your own Docker Registry.

**Github Packages now supports Docker Registry**

One of the easiest way to setup your own Docker Registry is by using [Github Packages](https://github.blog/2020-09-01-introducing-github-container-registry/).

Docker Registry in Github is also convenient. Having the Docker Images close to where you are storing your code allows for better developer productivity.

However setting up your first Docker Registry using Github Packages is not very well documented in my opinion. Here’s my attempt to **simplify your initial setup and save you time. **Read until the end for common errors and how to troubleshoot.

**TLDR**; Fix Docker Rate Limit issues by using Github Packages

## Steps

1. Create a Github Repo where you will store your docker image / Alternatively use an existing Github Repo.

![Create new Repo: https://github.com/kenichi-shibata/repo-for-images](https://cdn-images-1.medium.com/max/3484/1*uXkrbVgP0fX1kHQv2i8gsg.png)*Create new Repo: https://github.com/kenichi-shibata/repo-for-images*

2. Create a github token in [https://github.com/settings/tokens](https://github.com/settings/tokens)

![Click on Generate New Token](https://cdn-images-1.medium.com/max/2480/1*-ZixXQc9JHISIIH3uaB3qg.png)*Click on Generate New Token*

3. Setup your github token with write:packages repo and optionally delete:packages if you need it to automatically delete packages

![](https://cdn-images-1.medium.com/max/4548/1*NZzROynLn1XzfwTzVYaV2g.png)

4. Setup your .bashrc or .zshrc with environment variables or (export them manually)

<iframe src="https://medium.com/media/6eaf3d7880a81343273e6399a458b82b" frameborder=0></iframe>

5. login to github’s docker registry

<iframe src="https://medium.com/media/a41b0cfd7c1b4bef8dadf0efb6472775" frameborder=0></iframe>

6. To confirm you are logged in successfully try to tag and push nginx as a test image

<iframe src="https://medium.com/media/c36eef69467dd74ba9ca2838454e724a" frameborder=0></iframe>

7. view your docker image

![(its easy to miss check the **lower right side**)](https://cdn-images-1.medium.com/max/5712/1*q9ysVZOzaOz0d8IVDkgcyA.png)*(its easy to miss check the **lower right side**)*

![Versions are also available to be switched in the Recent Version tab Lower Right](https://cdn-images-1.medium.com/max/4864/1*9z769pd2bqHhBmjKWiHbmQ.png)*Versions are also available to be switched in the Recent Version tab Lower Right*

## Optional Steps

8. Pull your docker image. follow the instructions in the page

For example:

    docker pull docker.pkg.github.com/kenichi-shibata/repo-for-images/nginx:latest

9. delete your docker image via command

see [https://docs.github.com/en/free-pro-team@latest/packages/manage-packages/deleting-a-package#deleting-a-version-of-a-private-package-on-github](https://docs.github.com/en/free-pro-team@latest/packages/manage-packages/deleting-a-package#deleting-a-version-of-a-private-package-on-github)

## Troubleshooting:

    Error: Cannot perform an interactive login from a non TTY device

Make sure your GITHUB_TOKEN_RWD has a value

    unauthorized: Your token has not been granted the required scopes to execute this query. The 'id' field requires one of the following scopes: ['read:packages'], but your token has only been granted the: ['repo', 'workflow'] scopes. Please modify your token's scopes at: <https://github.com/settings/tokens>.

This means that you didn’t give the token the right privileges

    name unknown: The expected resource was not found.

The Github Repo does not exist. Each docker image corresponds to a github repo. It has a somehow convoluted convention.

    docker.pkg.github.com/OWNER/REPOSITORY/IMAGE_NAME:VERSION

whereas

* OWNER your usename or your org name

* REPOSITORY your actual github repository (this needs to exist)

* IMAGE_NAME this is changeable and it will be **created on push** if it does not exist

* VERSION docker version uses **latest** as default

so only IMAGE_NAME and VERSION can be changed

**TLDR;** create the github repo or fix the username

If you find any more errors feel free to comment so we can help out :) Next time we will discuss **how to build and push docker images using Github Actions and Github Packages as your Docker Registry.**

**References**

* [https://docs.github.com/en/free-pro-team@latest/packages/guides/configuring-docker-for-use-with-github-packages](https://docs.github.com/en/free-pro-team@latest/packages/guides/configuring-docker-for-use-with-github-packages)

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

👋 [**Join FAUN today and receive similar stories each week in your inbox!](https://faun.dev/join) **️ **Get your weekly dose of the must-read tech stories, news, and tutorials.**

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬

![](https://cdn-images-1.medium.com/max/3000/1*_cT0_laE4iPcqW1qrbstAg.gif)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
