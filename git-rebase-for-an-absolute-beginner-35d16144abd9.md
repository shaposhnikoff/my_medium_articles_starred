
# Git Rebase: For an Absolute Beginner



If you are a beginner to the Software Industry, version control would be one of the headaches that you come across at first. I myself thought I am familiar enough as a beginner with Git until I heard “First Rebase your repo with the master and then send a PR with your changes”.

I had worked with Git, did a few collaborative projects on Git with my friends but this is the first time I was working on a company project where I have to adhere to best practices and do it right. I struggled for sometime, deleted some of my work, tried different methods with no luck. I read some articles, watched some videos and ‘What Git Rebasing is’ was a very clear topic to me by then, but I still couldn't figure out the right way to do it. So here I am writing this post to make sure that you understand Git Rebasing very well and don’t have to go through all this trouble again. To make it easy to understand I will break this story into 2 posts. In this post, I will explain ‘What Git Rebasing Is” and in the next post I will explain how to do it right.

I must thank the WSO2 peeps who explained Git Rebasing to me very well (a few times actually — until I understood) and were patient with me for not knowing this basic process.

**What is Rebasing?**

If you are working in a repo where many number of people collaboratively work, the repo might change very quickly and it won’t be the same repo that you started to work on as new features/changes would have been added. This is where Rebasing comes into play. Rebasing takes your commits -> store it in a temporary location -> Take the latest version of your repo -> add your commits on top of the latest version available. So in this way Rebasing ensures that your commits are added to the current state of the project and not to the old state of the project which was there when you cloned the project

Too confusing? Never mind. I’ll walk you through what happens.

![Git Repo with master and another branch](https://cdn-images-1.medium.com/max/2000/1*nTlrFgyQMSdRwqU8lvKOHw.jpeg)*Git Repo with master and another branch*

This diagram shows a Git repo with master and another branch. Let's assume that you cloned this repo at this instance so that you have the latest version of the master branch.

![Git repo with a new branch created by you](https://cdn-images-1.medium.com/max/2000/1*e9fdETYhnkoWMVF-029ZWQ.jpeg)*Git repo with a new branch created by you*

Other people might be working on the existing branch but you cannot start on that branch as it contains unfinished work of others and therefore is not stable. So, you create a branch from master and start your work. After a few commits, your local repo will look like the above image.

![Git repo with the initial branch merged to master branch](https://cdn-images-1.medium.com/max/2000/1*4k-oilWFsiOsZOK1k7P3yw.jpeg)*Git repo with the initial branch merged to master branch*

Now let's say that the people who worked on the existing branch have finished their work and merged the code to the master branch of the original repo.

*Note: Here ‘original repo’ refers to the repo up-to-date repo available in Github and ‘local repo’ refers to the local copy of the repo available in your PC. ‘local repo’ may not be up-to-date as you need to manually update it. I have used these words to make this simple and understandable to a beginner.*

![Git repo which fails to merge the new branch as the master branch is now different](https://cdn-images-1.medium.com/max/2000/1*fOEFwk6OiJE7H7IMGEkmRg.jpeg)*Git repo which fails to merge the new branch as the master branch is now different*

Now if you try to merge your code into the master branch of the original repo without pulling its changes, you won’t be able to do that because the master in your local machine and the master in the original repo are at 2 different places. This is when you have to rebase your work with the master branch of the original repo.

![Rebasing your new branch with master](https://cdn-images-1.medium.com/max/2000/1*V5ZCgyFpiuKH2EC-rLLVvg.jpeg)*Rebasing your new branch with master*

Git Rebasing takes your commits and adds them on the latest version of the master branch available on the original repo(not the version which was there when you created the branch). First, you fetch the changes of the original repo to your local repo. Now your local master branch is up-to-date with the repo’s master branch. Then you should checkout to your branch and rebase it with the master branch. Now you can merge your branch to the master as your branch is based on the latest version of the master branch of the original repo.Then your branch will look as in the above image. After that, you can send your changes as a PR to the original repo to be merged. Once that is done,the original repo will look like the following image

![Git repo with the rebased new branch merged to the master](https://cdn-images-1.medium.com/max/2000/1*oaQIhoYtTEFt3had1tw4YQ.jpeg)*Git repo with the rebased new branch merged to the master*

To simplify the explanation, I have written about working on a repo directly without forking it, which won’t usually happen. When you work in a project you usually follow a Git Workflow. What you usually do is that you fork that repo first. Then clone the forked repo and do your work on it. This is a little bit advanced so I decided not to explain it here. In my next blog post I will explain about a Git Workflow which includes how to fork a repo and how to rebase it with the parent repo before pushing your changes as a PR to the parent repo.

*As I have said at the beginning of this post I am no expert on this because I just learnt it. I wrote this blog post so that a beginner can understand rebasing easily without having to know any Git workflows. So if there are any mistakes please be kind enough to let me know *😃

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
