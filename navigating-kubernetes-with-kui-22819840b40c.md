Unknown markup type 10 { type: [33m10[39m, start: [33m147[39m, end: [33m158[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m162[39m, end: [33m165[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m179[39m, end: [33m181[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m82[39m, end: [33m90[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m200[39m, end: [33m229[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m224[39m, end: [33m232[39m }

# Navigating Kubernetes with Kui

Kui is a graphical terminal — the CLI with a GUI twist. At its heart, Kui is a ASCII terminal, with a command line approach to interacting with your cloud resources. In response to certain commands, Kui augments the ASCII art presented by those commands with graphics. In this blog, I will highlight using Kui to interact, visually, with your Kubernetes cluster.

![**Kui** is at once both a plain ASCII art terminal and a modern browser](https://cdn-images-1.medium.com/max/2000/1*YyWzawiJBmvrxfXnegJTzA.gif)***Kui** is at once both a plain ASCII art terminal and a modern browser*

Above is a showcase of Kui in action. Observe how **everything I do is driven by a command line**. I start with the plain and familiar ASCII output of git branch and pwd. Next, I use ls to list the contents my current directory (which, in this case, happens to be the top-level of the [Kui source tree](https://github.com/IBM/kui)). The directory entries are rendered by Kui as **clickable links**.

And so, in this demo, I clicked away! Thankfully, Kui does not make me to beholden to my mouse; I could also have typed the name (with the assistance of tab completion) of the sub-directory I wanted to visit. For my daily habits, and this is coming from a 30-year veteran of the UNIX command line, having the option to click is one that I leverage often while using Kui — especially while giving demos or working with other team members.

![](https://cdn-images-1.medium.com/max/2000/1*yWCdERD0TOcIJDpQoPvwsg.png)

Next in my animated GIF demo, I switched from interacting with my local filesystem to interacting with my remote Kubernetes cluster. To the left is an expanded example of Kui’s **graphical tables**, here showing a list of my Kubernetes pods.

This Kui table is almost identical in structure to what you would see if you used kubectl directly. This was intentional: most command lines have put some good thought into what information to display; Kui’s job is not to improve on that aspect of the command line experience.

![Kui tables can also run in a live mode.](https://cdn-images-1.medium.com/max/2000/1*GetsKkfraTpw0jQ8v9Ynhg.gif)*Kui tables can also run in a live mode.*

Kui** enhances that basic ASCII picture**, using a rendering scheme (HTML5) that is richer than the fairly narrow palette offered by conventional terminals. For example, as shown on the left, Kui can render live updates to the table using some simple but helpful animations.

The final feature of Kui that I would like to introduce is the **sidecar**. When working with Kubernetes resources, it is common to spend a fair amount of time navigating between summary views (such as a list of pods) and detail views (such as the YAML definition of a resource, or the logs of a container).

![The Kui **sidecar** facilitates drilling down to the details of resources. When you are done exploring, the command line (on the left) remains unpolluted.](https://cdn-images-1.medium.com/max/2136/1*rTGhuQZc14X7NcgsCAsVDg.gif)*The Kui **sidecar** facilitates drilling down to the details of resources. When you are done exploring, the command line (on the left) remains unpolluted.*

Kui facilitates this style of ad hoc exploration by keeping the summary and detail views separate. When you request the details of a pod, for example (either by clicking on a table row, or by issuing kubectl get pod nginx -o yaml, Kui displays the details off to the side. You can see it sweeping in from the right in this animated GIF.

Every resource has multiple perspectives, and Kui allows you to flip between these quickly using the tabular structure of the sidecar. Note how I view the Last Applied Configuration (which would normally require some clever jsonpath and bash scripting), then I flip over to view the containers of this pod. From there, I can further drill down to the logs of a selected container.

Finally, observe how every drilldown not only manifests itself visually, but also the command to actuate this transition shows up in the command line. Kui tries, as much as possible, to remain a healthy spirit of transparency. It adds graphics where they count, but its heart is pure command line.

**Coming next**: “Visualizing CI/CD Pipelines in a Terminal”

Kui came from IBM Research, but is open to everyone. It is Apache-2.0 licensed, and has a powerful and flexible **extension **and **packaging** mechanism. [Join us](https://github.com/IBM/kui) on GitHub, or download a [**prebuilt binary](https://github.com/IBM/kui/blob/master/docs/installation.md)**.

*The entirely personal views of [Nick Mitchell](https://medium.com/the-graphical-terminal), nighttime hacker and daytime hiker.*
