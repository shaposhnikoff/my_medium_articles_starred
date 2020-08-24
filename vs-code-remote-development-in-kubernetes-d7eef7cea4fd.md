Unknown markup type 10 { type: [33m10[39m, start: [33m45[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m92[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m84[39m, end: [33m97[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m66[39m, end: [33m96[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m142[39m, end: [33m160[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m166[39m, end: [33m170[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m195[39m, end: [33m209[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m143[39m, end: [33m147[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m151[39m, end: [33m165[39m }

# VS Code Remote Development in Kubernetes



[VS Code Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack) is a powerful VS Code extension that allows you to take advantage of VS Code’s full feature set in the following scenarios:

* Develop a local folder in a local container using volume mounts.

* Develop a remote folder from a remote machine using SSH.

Development environments are getting more complex, in great part due to the broader variety of technologies being used today (e.g. polyglot apps, micro-service or third-party APIs). Instead of having to spend hours setting everything, VS Code Remote Development simplifies it by letting you use a pre-configured container as your development environment.

As teams have become more geographically distributed, a need for new collaboration models has arisen. In the middle of the Cloud Native revolution, we still develop locally. VS Code Remote Development is helping us evolve towards a Cloud Native development workflow.

In this blog post, I’ll explain the advantages of developing directly in a remote container running in Kubernetes, and how to achieve the best developer experience with the combined powers of VS Code Remote Development and Okteto.

**Why should you develop directly in Kubernetes?**

There are three important benefits of developing directly in Kubernetes:

* You are developing in a container: Containers give you replicable and isolated environments, without affecting your local machine.

* You are developing in Kubernetes: Develop with the same Kubernetes Manifests that in production, using the same ingress rules, secrets, config maps, volumes, service meshes or Admission Webhooks. Kubernetes will launch your development environments in seconds, scaling them vertically or horizontally to make the best usage of your resources.

* You can opt in to develop remotely: Remote execution gives you access to the same OS, network speed and hardware than in your cluster. It also enables a new level of collaboration by making your development environment available to the rest of your team.

Powerful, isn’t it? But what about my local development experience? As a developer, I love my debuggers, my VS Code extensions… I want to be fast, and not have to build a Docker image or call kubectl every time I want to validate my changes.

Let me show you how to keep this amazing development experience while working remotely through a sample application.

**Deploy the Hello World App to Kubernetes**

Get a local version of the Hello World App, by executing the following commands:

    $ git clone [https://github.com/okteto/go-getting-started](https://github.com/okteto/go-getting-started)
    $ cd go-getting-started

The Hello World App is written in Go and the k8s.yml file contains the Kubernetes manifests to deploy it. Run the application by executing:
> *You can deploy to your own Kubernetes cluster or give [Okteto Cloud](https://okteto.com/blog/how-to-develop-aspnetcore-apps-in-kubernetes/www.okteto.com) a try. Okteto Cloud is a development platform for Kubernetes applications. Sign up today to get a free developer account with 4CPUs and 8GB of RAM.*

    $ kubectl apply -f k8s.yml
    deployment.apps “hello-world” created
    service “hello-world” created

This is cool! You typed one command and a dev version of your application just runs 😎.

**Start your development environment in Kubernetes**

With the Hello World App deployed, run the following command:
> If you don’t have the Okteto CLI install, follow [this guide](https://okteto.com/docs/getting-started/installation/index.html).

    $ okteto up --remote 22000
    ✓ Development environment activated
    ✓ Files synchronized
      Namespace: pchico83
      Name: hello-world
      Forward: 8080 -> 8080
               22000 -> 22001

    Welcome to your development environment. Happy coding!
    okteto>

The okteto up command starts a [Kubernetes development environment](https://okteto.com/docs/reference/development-environment/index.html), which means:

* The Hello World App container is updated with the docker image okteto/hello-world:golang-dev. This image contains the required dev tools to build, test, debug and run the Go Sample App. Check the [Dockerfile](https://github.com/okteto/go-getting-started/blob/master/Dockerfile) to see how it is generated.

* A [file synchronization service](https://okteto.com/docs/reference/file-synchronization/index.html) is created to keep your changes up-to-date between your local filesystem and your application pods.

* Container port 8080 (the application) is forwarded to localhost, to make it simple to access your application.

* An [SSH server](https://github.com/okteto/remote) is started in your development environment and an entry added to your ~/.ssh/config file. This enables the integration between your development environment and the VS Code Remote SSH extension.

The next figure shows how these pieces are linked together:

![](https://cdn-images-1.medium.com/max/4600/0*K6SObjPkUzxHxbJE.png)

**Open VS Code and connect it to your Kubernetes development environment**

We’re ready to start developing in VS Code. Open VS Code, and run Remote-SSH: Connect to Host... from the Command Palette (F1) and select the hello-world.okteto entry:

![](https://cdn-images-1.medium.com/max/3428/0*Epwylc9Pl2VgQVSS.png)

After a few seconds, VS Code will connect over SSH and configure itself. Once it’s finished, you’ll be in an empty window. Click on the Open Folder button and select /app. After a few seconds, the remote folder will be loaded:

![](https://cdn-images-1.medium.com/max/3992/1*8j93GM4a34yI28Q5Um8low.png)

From now on, any actions you perform will happen directly in your Kubernetes development environment. Open a new terminal window in VS Code (Terminal > New Terminal) and run the application with go run main.go:

![](https://cdn-images-1.medium.com/max/4180/0*g9p2tRpRg6jNaZwG.png)

Open your browser and navigate to localhost:8080 to access your application. You can access it locally because okteto forwards the remote port 8080 to localhost:8080. Install your favorite extensions like debuggers, test frameworks, linters, … you can use all of them in your Kubernetes development environment as if you were working locally. Okteto uses a persistent volume to keep your extensions already installed the next time you activate your Kubernetes development environment 😁.
> We recommend to run Git extensions locally. This way they use your local keys and you don´t need to install them remotely.

**Conclusions**

We explained the advantages of developing directly in Kubernetes while keeping the same developer experience than working on a local machine. Working on the Cloud is always better. You don’t work on your spreadsheets and listen to media files locally, do you? Stop dealing with local environments and [become a Cloud Native Developer](https://okteto.com/) today 😎!
