
# Triggering Jenkins build on push using GitHub webhooks

Jenkins Github Integration
> **Jenkins** is a popular open source tool to perform continuous integration and build automation. Let’s take a look at how we can integrate **GitHub **with Jenkins for** source code management** and **trigger build on push using web-hooks**.

### Prerequisites :

1. **Jenkins**: Download and install Jenkins as described [*here](https://jenkins.io/download/).*

1. **Git**: Install Git. To check whether you have git installed, open a terminal window and type below command.

    git --version

3. **Plugins: **Add Git and GitHub Plugins.

**Go to Manage-Jenkins-> Manage Plugin**

search ***Github plugin*** and install without restart

![Plugin Manager](https://cdn-images-1.medium.com/max/3200/0*hJs3xncPq6ltul01.jpg)*Plugin Manager*

**4. Go to Manage Jenkins -> Global Tool Configuration -> Git**

Add git executable path to Global Tool Configuration.

![Global tool configurations](https://cdn-images-1.medium.com/max/3740/1*AJ_vb37b6zNxrk3gyYd6iA.png)*Global tool configurations*

Let us start with creating a Freestyle Project :

**Step 1:** **Go to New Item -> create a freestyle project**.

![](https://cdn-images-1.medium.com/max/3278/1*IJLSdbt-Ia-BS3pjWdJPUQ.png)

![Creating a freestyle project](https://cdn-images-1.medium.com/max/2136/1*OOzt0LNzxDZd8o5uTGuM0A.png)*Creating a freestyle project*

**Step 2:** Go to **Configure, **add a project description, and Github project URL.

**Step 3:** In** Source Code Management **tab select on **Git**, add your Github repository URL and click Add button to save your GitHub credentials.

**Save and click build **to make sure everything is right till here and your project is successfully building.

**step 4:** In **Build Triggers** select **GitHub hook trigger for GITScm polling.** When Jenkins will receive PUSH GitHub hook, it will trigger Git SCM polling logic which will start a new Jenkins build, with the updated code.

![Build Triggers](https://cdn-images-1.medium.com/max/2936/1*GDpDlSZEVy9qMnBU5Pryjg.png)*Build Triggers*
> We use [***ngrok](https://ngrok.com/download) ***to expose local jenkins to the internet, so that github can send the webhooks.

**Step 5:** **Go to your Github repo -> settings -> webhooks**

Add public URL of your tunnel as **Payload URL, **it will tell Github where to send the webhooks as below:

    https://0584fb6b.ngrok.io/github-webhook/

![](https://cdn-images-1.medium.com/max/3146/1*jdYBxbCCVpr2aT4yzpONBw.png)

**step 6: **Finally** **add your build steps in the **Build** tab and save.

That’s all! Now whenever any change is pushed, a new Jenkins build will be triggered.

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Join our community Slack and read our weekly Faun topics ⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
