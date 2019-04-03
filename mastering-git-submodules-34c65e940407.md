
# Mastering Git submodules

If you used submodules before, you certainly got a few scars to show for it, probably swearing off the dang thing. Submodules are hair-pulling for sure, what with their host of pitfalls and traps lurking around most use cases. Still, they are not without merits, if you know how to handle them.

In this post, we’ll dive deep into Git submodules, starting by making sure they’re the right tool for the job, then going through every standard use case, step by step, so as to illustrate best practices.

Submodules, like subtrees, aim to **reuse code from another repo somewhere inside your own repo’s tree**. The goal is usually to benefit from **central maintenance** of the reused code across a number of container repos, without having to resort to clumsy, unreliable copy-pasting.

In the remainder of this text, I’ll call such reused code, present somewhere inside container repo trees, a **“module.” **As for project code that reuses said module somewhere inside its working directory’s tree, I’ll call that a **“container.”**

## Are they the right tool for the job?

There are a number of situations where the physical presence of module code inside container code is mandated, usually because of the technology or framework being used. For instance, themes and plugins for Wordpress, Magento, etc. are often*** **de facto* installed by their mere presence at conventional locations inside the project tree, and this is the only way to “install” them.

In such a situation, going with submodules (or subtrees) probably is the right solution, provided you do need to version that code and collaborate around it with third parties (or deploy it on another machine); for strictly local, unversioned situations, symbolic links are probably enough, but this is not what this post is about.

On the other hand, if the technological context allows for *packaging* and formal dependency management, you should absolutely go this route instead: it lets you better split your codebase, avoid a number of side effects and pitfalls that litter the submodule space, and let you benefit from versioning schemes such as *semantic versioning* ([semver](http://semver.org/)) for your dependencies.
> If the technological context allows for *packaging *and formal dependency management, you should absolutely go this route.

As a reminder, here’s a list of the main languages and their dependency management / packaging systems and registries:

* Clojure has [Clojars](http://clojars.org/)

* Erlang has [Hex](https://hex.pm/)

* Go has [GoDoc](http://godoc.org/-/index) *(closest thing it has to a package registry, anyway)*

* Haskell has [Hackage](http://hackage.haskell.org/)

* Java has [Maven Central](http://www.maven.org/)

* JavaScript has [npm](https://www.npmjs.com/) and [Bower](http://bower.io/) (and others, too)

* .NET has [nuget](http://nuget.org/)

* Objective-C has [CocoaPods](http://cocoapods.org/)

* Perl has [CPAN](http://www.cpan.org/)

* PHP has [Composer](https://getcomposer.org/), [Packagist](http://packagist.org/) and good ol’ [Pear](http://pear.php.net/packages.php)

* Python has [PyPI](http://pypi.python.org/pypi)

* Ruby has [Bundler](http://bundler.io/) and [Rubygems](http://rubygems.org/)

* Rust has [Crates](https://crates.io/)

Honestly, if you can manage your code dependencies by packaging reused code cleanly in “centralized” modules and using dependency management tools, do it. For real. Honest. This will save you a world of pain (and you don’t necessarily have to publish your packages out in the open, these systems often allow for private packages too).

Still, if you have a solid requirement to embed reused code right inside the container code, then you are left with a choice between submodules and subtrees.

## Submodules or subtrees?

In general, [**subtrees are better](https://medium.com/p/mastering-git-subtrees-943d29a798ec)**. Hey, I’m doing a bang-up job of selling you this post, aren’t I? The fact is that submodules and subtrees are radically different, almost opposite in fact, be it in their concepts or their behavior.

Most people go with submodules for a few common reasons. Submodules have been around for a good long while, have their own Git command (*git submodule*), detailed docs, and a behavior not entirely unlike Subversion externals, which makes them feel falsely familiar. Adding a submodule is very simple (a quick *git submodule add*), especially compared to adding a subtree. Only later do all the pitfalls and traps come and bite *everyone, every day*.

It’s precisely because submodules have caused so many poor unsuspecting Gitters pain that we chose to cover them first, and subtrees later (our [next in-depth article](https://medium.com/@porteneuve/mastering-git-subtrees-943d29a798ec)).

Still, **sometimes, submodules are the right choice**. It’s especially true when your codebase is massive and you don’t want to have to fetch it all every time, a situation many tentacular code bases grapple with. You then resort to submodules so your collaborators don’t necessarily have to fetch entire blocks of the code base. Various open-source projects use submodules for precisely that reason (or because of heavy modularization not natively handled by their main language’s ecosystem).

You should also strive for submodule code to remain **independent of particularities of the container** (or at least, rely on external configuration to handle such particularities), as submodule code is central code, shared across all container projects. Working around this by littering your submodule repo with container-specific branches is like opening Pandora’s box: it’s abusive coupling, going against modularization and encapsulation principles, and is sure to come back and bite your ankle at some point.
> # Using Git with **GitHub**? Want to become a true GitHub master? We released part 1 of our best-of-class GitHub video training series! 5 hours, 69 videos, amazing contents for beginners and experts alike! [Learn more](https://medium.com/@porteneuve/our-github-video-course-series-is-out-1fe829e04a59).

## Submodule fundamentals

**A quick reminder of terminology first: with Git, a repo is local**. The remote version, which is mostly use for archiving, collaboration, sharing, and CI triggers, is called a *remote*. In the remainder of this text, whenever you read “repo” or “Git repo”, remember it’s your local, interactive repo (that is, with a working directory alongside its *.git* root).

Submodules rely on **nesting repos**: you have repos within… repos. The module has its own repo, somewhere inside the working directory of its container repo.

In practice, since Git 1.7.8, submodules use a simple *.git file* with a single *gitdir:* line mentioning a relative path to the actual repo folder, now located inside the container’s *.git/modules*. This is mostly useful when the container has branches that don’t have the submodule at all: this avoid having to scrap the submodule’s repo when switching to such a container branch.

Be that as it may, the container and the submodule truly act as independent repos: they each have their own history (log), status, diff, etc. **Therefore be mindful of your current directory when reading your prompt or typing commands**: depending on whether you’re inside the submodule or outside of it, the context and impact of your commands differ drastically!
> The container and the submodule truly act as independent repos.

Finally, **the submodule commit referenced by the container is stored using its SHA1**, not a volatile reference (such as a branch name). Because of this, **a submodule does not automatically upgrade** which is a **blessing** in disguise when it comes to reliability, maintenance and QA (just ask Subversionians using externals how many times they have to specify *--ignore-externals* in their commands to avoid untimely upgrades…).

Because of this, most of the time a submodule is in **detached head** state inside its containers, as it’s updated by checking out a SHA1 (regardless of whether that commit is the branch tip at that time).

## A plethora of traps

Submodules are a weird balance: **easy to set up, fraught with peril the entire rest of the project’s life**: whereas a simple *git submodule add* suffices to set one up, all contributors to the repo will from then on have to be especially watchful to avoid issues.

![iOS sometimes cunningly autocorrects submodules as “sobmodules.” Someone got bitten, it seems…](https://cdn-images-1.medium.com/max/2000/1*MAPuqWxvWMNrrwHMDg1NVw.png)*iOS sometimes cunningly autocorrects submodules as “sobmodules.” Someone got bitten, it seems…*

Obviously, this is a dangerous approach. Because **constant vigilance does not work. Ever.** It’s no accident, I guess, that autocomplete on iOS sometimes suggests “sobmodules” about them.

In this article, during the upcoming step-by-step, we’ll detail what the pitfalls are, and what can be done to alleviate or even neutralize most of them, through CLI options or configuration settings.

But for now, let’s quickly list what traps there are.

### The dangers we face

* Every time you add a submodule, change its remote’s URL, or change the referenced commit for it, you demand** a manual update by every collaborator**.

* Forgetting this explicit update can result in **silent regressions** of the submodule’s referenced commit.

* Commands such as *status *and *diff* display **precious little info** about submodules by default.

* Because lifecycles are separate, updating a submodule inside its container project requires **two commits and two pushes**.

* Submodule heads are generally detached, so any local update requires various **preparatory actions** to avoid creating a lost commit.

* Removing a submodule requires several commands and tweaks, some of which are **manual and unassisted**.

In short, folks, we’ll need to **know what we’re doing**.

## Submodules, step by step

We’ll now explore every step of using submodules in a collaborative project, making sure we highlight **default behaviors**, **traps** and available **improvements**.

In order to facilitate your following along, I’ve put together a few **example repos** with their “remotes” (actually just directories). You can uncompress the archive wherever you want, then open a shell (or Git Bash, if you’re on Windows) in the *git-subs* directory it creates:
<center>
[**Download the example repos](http://drive.delicious-insights.com/assets/git-subs-demo.zip)**</center>

You’ll find three directories in there:

* *main* acts as the **container** repo, local to the first collaborator,

* *plugin* acts as the **central maintenance repo** for the module, and

* *remotes* contains the filesystem remotes for the two previous repos.

In the example commands below, the prompt always displays which repo we’re into.

### Adding a submodule

Let’s start by adding our plugin as a submodule inside our container (which is in *main*). The plugin itself has a simple structure:

    .
    ├── README.md
    ├── lib
    │   └── index.js
    └── plugin-config.json

So let’s go into *main* and use the *git submodule add* command. It takes the remote’s URL and a subdirectory in which to “instantiate” the submodule.

Because we use paths instead of URLs here for our remotes, we hit a weird, albeit well-known, snag: relative paths for remotes are interpreted relative to our main remote, no to our repo’s root directory. This is super weird, not described anywhere, but I’ve seen it happen every time. So instead of saying *../remotes/plugin*, we just say *../plugin*.

    main (master u=) $ git submodule add ../plugin vendor/plugins/demo
    Cloning into 'vendor/plugins/demo'…
    done.
    main (master + u=) $

This added some settings in our local configuration:

    main (master + u=) $ cat .git/config
    …
    [submodule "vendor/plugins/demo"]
      url = ../remotes/plugin

And this also staged **two** files:

    main (master + u=) $ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Changes to be committed:
      (use "git reset HEAD <file>..." to unstage)

      new file: .gitmodules
      new file: vendor/plugins/demo

Huh?! What’s this *.gitmodules* file? Let’s look at it:

    main (master + u=) $ cat .gitmodules
    [submodule "vendor/plugins/demo"]
      path = vendor/plugins/demo
      url = ../plugin

This furiously resembles our local config… So why the duplication? Well, precisely because our local config is… local. Our collaborators won’t see it (which is perfectly normal), so they need a mechanism to **get the definitions of all submodules they need to set up in their own repos**. This is what *.gitmodules* is for; it will be read later by the *git submodule init* command, as we’ll see in a moment.

While we’re on status, note how **minimalistic** it is when it comes to our submodule: it just goes with an overly generic *new file* instead of telling us more about what’s going on inside it. Our submodule was indeed injected in the subdirectory:

    …
    └── vendor
        └── plugins
            └── demo
                ├── .git
                ├── README.md
                ├── lib
                │   └── index.js
                └── plugin-config.json

Status, like logs and diffs, is limited to the **active repo** (right now, the container), not to submodules, which are nested repos. This is often problematic (it’s super easy to miss a regression when limited to this view), so I recommend you **set up a submodule-aware status once and for all**:

    git config --global status.submoduleSummary true

And now:

    main (master + u=) $ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Changes to be committed:
      (use "git reset HEAD <file>..." to unstage)

      new file: .gitmodules
      new file: vendor/plugins/demo

    Submodule changes to be committed:

    * vendor/plugins/demo 0000000...fe64799 (3):
      > Fix repo name for main project companion demo repo

Aaaah, this is vastly better. The status extends its base information to add that the submodule present at *vendor/plugins/demo* got 3 news commits in (as we just created it, it means the remote branch had only three commits), the last one being an addition (note the right-pointing angle bracket *>*) with a first commit message line that reads “Fix repo name…”.

In order to really bring home that we deal with **two separate repos** here, let’s get into the submodule’s directory:

    main (master + u=) $ cd vendor/plugins/demo
    demo (master u=) $ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    nothing to commit, working directory clean

**The active repo has changed**, because a new *.git* takes over: in the current directory (*demo*, the submodule’s directory), a *.git* exists indeed, a single **file** too, not a directory. Let’s look inside:

    demo (master u=) $ cat .git
    gitdir: ../../../.git/modules/vendor/plugins/demo

Again, since Git 1.7.8, Git does not leave repo directories inside the container’s working directory, but centralizes these in the container’s *.git* directory (inside *.git/modules*), and uses a *gitdir* reference in submodules.

The rationale behind this is simple: it allows the **container repo to have submodule-less branches**, without having to scrap the submodule’s repo from the working directory and restoring it later.

Naturally, when adding the submodule, you can elect to use a specific branch, or even a specific commit, using the *-b* CLI option (as per usual, the default is *master*). Note we’re not, right now, on a detached head, unlike what will happen later: this is because Git checked out *master*, not a specific SHA1. We would have had to specify a SHA1 to *-b* to get a detached head from the get-go.

So, back to the container repo, and let’s finalize the submodule’s addition and push that to the remote:

    demo (master u=) $ cd -
    main (master + u=) $ git commit -m "Ajout submodule plugin demo"
    main (master u+1) $ git push

### Grabbing a repo that uses submodules

In order to illustrate the issues with **collaborating** on a repo that uses submodules, we’ll split personalities and act as our colleague, who clones the container’s remote to start working with us. We’ll clone that in a *colleague* directory, so we can immediately tell which personality cap we have on at any given time.

    main (master u=) $ cd ..
    git-subs $ git clone remotes/main colleague
    Cloning into 'colleague'...
    done.
    git-subs $ cd colleague
    colleague (master u=) $

The first thing to notice is that **our submodule is missing** from the working directory; only its base directory is here:

    vendor
    └── plugins
        └── demo

How did that happen? This is simply due to the fact that, so far, our new repo (*colleague*) **is not aware of our submodule yet**: the information for it is nowhere in its local configuration (check its *.git/config* if you don’t believe me). We’ll need to fill that in, based on what *.gitmodules* has to say, which is precisely what *git submodule init* does:

    colleague (master u=) $ git submodule init
    Submodule 'vendor/plugins/demo' (/tmp/git-subs/remotes/plugin) registered for path 'vendor/plugins/demo'

Our *.git/config* is now aware of our submodule. However, we **still haven’t fetched it** from its remote, to say nothing of having it present in our working directory. And yet, our status shows up as clean!

See, we need to grab the relevant commits manually. It’s not something our initial *clone* did, we need to do it on every pull. We’ll come back to that in a minute, as this is a behavior *clone* can actually automate, when properly called.

    colleague (master u=) $ git submodule update
    Cloning into 'vendor/plugins/demo'...
    done.
    Submodule path 'vendor/plugins/demo': checked out 'fe6479991d214f4d95ac2ae959d7252a866e01a3'

In practice, when dealing with submodule-using repos, we usually **group** the two commands (init and update) in one:

    colleague (master u=) $ git submodule update --init

It is still a shame that Git has you do all that yourself. Just imagine, on larger FLOSS projects, when submodules have their own submodules, and so forth and so on… This would quickly become a nightmare.

It so happens that **Git does provide a CLI option** for *clone* to automatically *git submodule update --init* recursively right after cloning: the rather aptly-named *--recursive* option.

So let’s try the whole thing again:

    colleague (master u=) $ cd -
    git-subs $ rm -fr colleague
    git-subs $ **git clone --recursive remotes/main colleague**
    Cloning into 'colleague'...
    done.
    Submodule 'vendor/plugins/demo' (/tmp/git-subs/remotes/plugin) registered for path 'vendor/plugins/demo'
    Cloning into 'vendor/plugins/demo'...
    done.
    Submodule path 'vendor/plugins/demo': checked out 'fe6479991d214f4d95ac2ae959d7252a866e01a3'

Now that’s better! Note that we’re now on a detached head inside the submodule (as we’ll be from now on):

    git-subs $ cd colleague/vendor/plugins/demo
    demo ((master)) $

See the double set of parentheses in my prompt, instead of a single set? If your prompt is not configured like mine, to display detached head as *describes* (with Git’s built-in prompt script, you’d have to define the *GIT_PS1_DESCRIBE_STYLE=branch* environment variable), you’ll rather see something like this:

    demo ((fe64799...)) $

At any rate, *status* confirms where we’re at:

    demo ((master)) $ git status
    HEAD detached at fe64799
    nothing to commit, working directory clean

### Getting an update from the submodule’s remote

OK, now that we have our own repo (*main*) and our “colleague’s” (*colleague*) all set up to collaborate, let’s step into the shoes of a **third person**: the one who maintains the plugin. Here, let’s move to it:

    colleague (master u=) $ cd ../plugin
    plugin (master u=) $ git log --oneline
    fe64799 Fix repo name for main project companion demo repo
    89d24ad Main files (incl. subdir) for plugin, to populate its tree.
    cc88751 Initial commit

Now, let’s add two pseudo-commits and publish these to the remote:

    plugin (master u=) $ date > fake-work
    plugin (master % u=) $ git add fake-work
    plugin (master + u=) $ git commit -m "Pseudo-commit #1"
    [master e6f5bb6] Pseudo-commit #1
     1 file changed, 1 insertion(+)
     create mode 100644 fake-work

    plugin (master u+1) $ date >> fake-work
    plugin (master * u+1) $ git commit -am "Pseudo-commit #2"
    …
    plugin (master u+2) $ git push

Finally, let’s put our “**first developer**” cap on again:

    plugin (master u=) $ cd ../main
    main (master u=) $

Suppose we now want to **get these two commits inside our submodule**. To achieve this, we need to **update its local repo**, starting by moving into its working directory so it becomes our active repo.

On a side note, **I would not recommend using *pull*** for this kind of update. To properly get the updates in the working directory, this command requires that you’re on the proper active branch, which you usually aren’t (you’re on a detached head most of the time). You’d have to start with a *checkout* of that branch. But more importantly, the remote branch could very well have moved further ahead since the commit you want to set on, and a *pull* would inject commits you may not want in your local codebase.

Therefore, I recommend **splitting the process** manually: first *git fetch* to get all new data from the remote in local cache, then *log* to verify what you have and *checkout* on the desired SHA1. In addition to **finer-grained control**, this approach has the added benefit of working regardless of your current state (active branch or detached head).

    main (master u=) $ cd vendor/plugins/demo
    demo (master u=) $ **git fetch**
    remote: Counting objects: 6, done.
    remote: Compressing objects: 100% (5/5), done.
    remote: Total 6 (delta 1), reused 0 (delta 0)
    Unpacking objects: 100% (6/6), done.
    From /private/tmp/git-subs/main/../remotes/plugin
       fe64799..0e90143 master -> origin/master

    demo (master u-2) $ git log --oneline origin/master -10
    0e90143 Pseudo-commit #2
    e6f5bb6 Pseudo-commit #1
    fe64799 Fix repo name for main project companion demo repo
    89d24ad Main files (incl. subdir) for plugin, to populate its tree.
    cc88751 Initial commit

OK, so we’re good, no extraneous commit. Be that as it may, let’s explicitly set on the one we’re interested in (obviously you have a different SHA1):

    demo (master u-2) $ git checkout -q 0e90143

(The *-q* is only there to spare us Git blabbering about how we’re ending up on a detached head. Usually this would be a healthy reminder, but on this one we know what we’re doing.)

Now that our submodule is updated, we can see the result in the **container repo’s status**:

    demo ((remotes/origin/HEAD)) $ cd -
    main (master * u=) $ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
      (use "git checkout -- <file>..." to discard changes in working directory)

      modified: vendor/plugins/demo **(new commits)**

    Submodules changed but not updated:

    * vendor/plugins/demo fe64799...0e90143 **(2)**:
      **> Pseudo-commit #2
      > Pseudo-commit #1**

    no changes added to commit (use "git add" and/or "git commit -a")

In the “classical” part of the status, we see a *new commits* change type, which means **the referenced commit changed**. Another possibility (which could be compounded to this one) is *new contents*, which would mean we made local changes to the submodule’s working directory.

The lower part, enabled by our *status.submoduleSummary = true* setting earlier on, explicitly states the **introduced commits** (as they use a right-pointing angle bracket *>*) since our last container commit that had touched the submodule.

In the “terrible default behaviors” family, *git diff* leaves a lot to be desired:

    main (master * u=) $ git diff
    diff --git i/vendor/plugins/demo w/vendor/plugins/demo
    index fe64799..0e90143 160000
    --- i/vendor/plugins/demo
    +++ w/vendor/plugins/demo
    @@ -1 +1 @@
    -Subproject commit fe6479991d214f4d95ac2ae959d7252a866e01a3 +Subproject commit 0e9014309fe6c663e806c9f91297a592ee04cb6c

What the — ? There’s a CLI option that lets us see something more useful:

    main (master * u=) $ git diff **--submodule=log**
    Submodule vendor/plugins/demo fe64799..0e90143:
      > Pseudo-commit #2
      > Pseudo-commit #1

There are no other local changes right now besides the submodule’s referenced commit… Notice this matches almost exactly the lower part of our enhanced *git status* display.

Having to type that kind of CLI option every time (which, by the way, does not show up in Git’s current completion offers) is rather unwieldy. Fortunately, there is a matching **configuration setting**:

    **git config --global diff.submodule log**

    main (master * u=) $ git diff
    Submodule vendor/plugins/demo fe64799..0e90143:
     > Pseudo-commit #2
     > Pseudo-commit #1

We now only need to perform **the container commit that finalizes our submodule’s update**. If you had to touch the container’s code to make it work with this update, commit it along, naturally. On the other hand, **avoid mixing submodule-related changes and other stuff** that would just pertain to the container code: by neatly separating the two, later migrations to other code-reuse approaches are made easier (also, as usual, atomic commits FTW).

As we’re about to grab this submodule update in our colleague’s repo, we’ll push right after committing (which is *not* a general good practice).

    main (master * u=) $ git commit -am "Setting submodule on PC2"
    main (master u+1) $ git push

### Pulling a submodule-using repo

Click! “Colleague” cap on!

So we’re pulling updates from the container repo’s remote…

    colleague (master u=) $ git pull
    remote: Counting objects: 4, done.
    remote: Compressing objects: 100% (2/2), done.
    remote: Total 4 (delta 1), reused 0 (delta 0)
    Unpacking objects: 100% (4/4), done.
    From /tmp/git-subs/remotes/main
       c995ed0..ac96c22 master -> origin/master
    **Fetching submodule vendor/plugins/demo**
    remote: Counting objects: 6, done.
    remote: Compressing objects: 100% (5/5), done.
    remote: Total 6 (delta 1), reused 0 (delta 0)
    Unpacking objects: 100% (6/6), done.
    From /tmp/git-subs/remotes/plugin
       fe64799..0e90143 master -> origin/master
    Successfully rebased and updated refs/heads/master.
    colleague (master * u=) $

*(You might not have the “Successfully rebased and updated…” and see a “Merge made by the ‘recursive’ strategy” instead. If so, my heart goes out to you, and you should immediately [learn why pulls should rebase](https://medium.com/@porteneuve/getting-solid-at-git-rebase-vs-merge-4fa1a48c53aa#7219)).*

Note the second half of this display: it’s about the submodule, starting with *“Fetching submodule…”*.

This behavior became the default with Git 1.7.5, with the configuration setting *fetch.recurseSubmodules* now defaulting to *on-demand*: if a container project gets updates to referenced submodule commits, these submodules get fetched automatically. (Remember fetching is the first part of pulling.)

Still, and this is critical: **Git auto-fetches, but does not auto-update**. Your local cache is up-to-date with the submodule’s remote, but the submodule’s **working directory stuck to its former contents**. At least, you can shut that laptop, hop onto a plane, and still forge ahead once offline. Although this auto-fetching is limited to already-known submodules: any new ones, not yet copied into local configuration, are not auto-fetched.
> Git auto-fetches, but does not auto-update.

The current prompt, with its asterisk (**)*, does hint at local modifications, because our WD is not in sync with the index, the latter being aware of the newly referenced submodule commits. Check out the status:

    colleague (master * u=) $ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
      (use "git checkout -- <file>..." to discard changes in working directory)

      modified: vendor/plugins/demo (new commits)

    Submodules changed but not updated:

    * vendor/plugins/demo 0e90143...fe64799 **(2)**:
    **  < Pseudo-commit #2
      < Pseudo-commit #1**

    no changes added to commit (use "git add" and/or "git commit -a")

Notice how **the angle brackets point left** (*<*)? Git sees that the current WD does not have these two commits, contrary to the container project’s expectations.

**This is the massive danger: if you don’t explicitly update the submodule’s working directory, your next container commit will regress the submodule**. This is a first-order trap.

Is is therefore **mandatory** that you finalize the update:

    colleague (master * u=) $ git submodule update
    Submodule path 'vendor/plugins/demo': checked out '0e9014309fe6c663e806c9f91297a592ee04cb6c'

As long as we’re trying to form generic good habits, the preferred command here would be a *git submodule update --init --recursive*, in order to auto-init any new submodule, and to recursively update these if need be.

There is another edge case: if the submodule’s remote URL changed since last used (perhaps one of the collaborators changed it in the *.gitmodules*), you have to manually update your local config to match this. In such a situation, *before* the *git submodule update*, you’d need to run a *git submodule sync*.

I should mention, for completeness’ sake, that even if *git submodule update* defaults to checking out the referenced SHA1, you can change that to, for instance, rebase any local submodule work (we’ll talk about that very soon) on top of it. You’d do that by setting the *update* configuration setting for your submodule to *rebase* inside your container’s local configuration.

And I’m sorry but no, **there’s no local configuration setting, or even CLI option for that matter, that can auto-update on *pull***. To automate such things, you’d need to use either aliases, custom scripts, or carefully crafted local hooks. Here’s an example *spull* alias (single line, split here for display):

    git config --global alias.spull '!git pull && git submodule sync --recursive && git submodule update --init --recursive'

If you want to retain the ability to pass custom arguments to *git pull*, you can either define a function on-the-fly and call it, or go with a custom script. The first approach would look like this (again, single line):

    git config --global alias.spull '__git_spull() { git pull "$@" && git submodule sync --recursive && git submodule update --init --recursive; }; __git_spull'

Not very readable, eh? I prefer the custom script approach. Let’s say you’d put a *git-spull* script file somewhere inside your *PATH* (I have a *~/perso/bin* directory in my *PATH* just for such things):

    *#! /bin/bash
    *git pull "$@" **&&
      **git submodule sync --recursive **&&
      **git submodule update --init --recursive

We then give it execution rights:

    chmod +x git-spull

And now we can use it just like we’d have used the alias.

### Updating a submodule in-place in the container

This is the **hardest use-case**, and you should stay away from it as much as possible, preferring maintenance through the central, dedicated repo.

However, it can happen that **submodule code cannot be tested, or even compiled, outside container code**. Many themes and plugins have such constraints.

The first thing to understand is, because you’re going to make commits, **you must start from a proper basis, which will be a branch tip**. You therefore need to verify that the branch’s latest commits won’t “break” your container project. If they do, well, creating your own container-specific branch in the submodule sounds tempting, but that path leads to **strong coupling** between submodule and container, which is not advisable. You may want to stop “submoduling” that code in this particular project, and just embed it like any regular contents instead.

Let’s admit that you can, in good conscience, add to the submodule’s current *master* branch. Let’s start by syncing our local state on the remote’s:

    colleague (master u=) $ cd vendor/plugins/demo
    demo ((remotes/origin/HEAD)) $ git checkout master
    Previous HEAD position was 0e90143... Pseudo-commit #2
    Switched to branch 'master'
    Your branch is behind 'origin/master' by 2 commits, and can be fast-forwarded.
      (use "git pull" to update your local branch)

    demo (master u-2) $ git pull --rebase
    First, rewinding head to replay your work on top of it...
    Fast-forwarded master to 0e9014309fe6c663e806c9f91297a592ee04cb6c.
    demo (master u=) $

Another way to go about this would be, from the container repo, to explicitly sync the submodule’s local branch over its tracked remote branch (single line on top, last *--* followed by whitespace):

    colleague (master u=) $ git submodule update --remote --rebase -- vendor/plugins/demo
    …

    colleague (master u=) $ cd vendor/plugins/demo
    demo (master u=) $

We can now edit the code, make it work, test it, etc. Once we’re all set, we can then perform **the two commits and the two necessary pushes** (it’s super easy, and in practice all too frequent, to forget some of that).

Let’s simply add fake work and make the two related commits, at the submodule and container levels:

    demo (master u=) $ date >> fake-work
    demo (master * u=) $ git commit -am "Pseudo-commit #3"
    [master 12e3a52] Pseudo-commit #3
     1 file changed, 1 insertion(+)

    demo (master u+1) $ cd ../../..
    colleague (master * u=) git commit -am "Using PC3 on the submodule"
    [master ad9da82] Using PC3 on the submodule
     1 file changed, 1 insertion(+), 1 deletion(-)
    colleague (master u+1) $

At this point, **the major danger is forgetting to push the submodule**. You get back to the container project, commit it, and only push the container. It’s an easy mistake to make, especially inside an IDE or GUI. When your colleagues try to get updates, all hell breaks loose. Look at the first step:

    colleague (master u+1) $ git push
    Counting objects: 4, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (2/2), done.
    Writing objects: 100% (4/4), 355 bytes | 0 bytes/s, done.
    Total 4 (delta 1), reused 0 (delta 0)
    To /tmp/git-subs/remotes/main
       766cd47..ad9da82 master -> master

    colleague (master u=) $ cd ../main
    main (master u=) $ git pull
    remote: Counting objects: 4, done.
    remote: Compressing objects: 100% (2/2), done.
    remote: Total 4 (delta 1), reused 0 (delta 0)
    Unpacking objects: 100% (4/4), done.
    From ../remotes/main
       766cd47..ad9da82 master -> origin/master
    **Fetching submodule vendor/plugins/demo**
    Successfully rebased and updated refs/heads/master.

    main (master * u=) $

There is **absolutely no indication that Git could not fetch** the referenced commit from the submodule’s remote. The first hint of this is in the status:

    main (master * u=) $ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
      (use "git checkout -- <file>..." to discard changes in working directory)

      modified: vendor/plugins/demo (new commits)

    Submodules changed but not updated:

    * vendor/plugins/demo 12e3a52...0e90143:
      **Warn: vendor/plugins/demo doesn't contain commit 12e3a529698c519b2fab790630f71bd531c45727**

    no changes added to commit (use "git add" and/or "git commit -a")

Notice the warning: apparently, the newly referenced commit for the submodule is nowhere to be found. Indeed, if we attempt updating the submodule’s working directory, we get:

    main (master * u=) $ git submodule update
    fatal: reference is not a tree: 12e3a529698c519b2fab790630f71bd531c45727
    Unable to checkout '12e3a529698c519b2fab790630f71bd531c45727' in submodule path 'vendor/plugins/demo'

You can plainly see how important it is to **remember pushing the submodule too**, ideally *before* pushing the container. Let’s do that in *colleague* and attempt the update again:

    main (master * u=) $ cd ../colleague/vendor/plugins/demo
    demo (master u+1) $ git push
    Counting objects: 3, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (3/3), done.
    Writing objects: 100% (3/3), 329 bytes | 0 bytes/s, done.
    Total 3 (delta 1), reused 0 (delta 0)
    To /tmp/git-subs/remotes/plugin
       0e90143..12e3a52 master -> master

    demo (master u=) $ cd -
    main (master * u=) $ git submodule update
    remote: Counting objects: 3, done.
    remote: Compressing objects: 100% (3/3), done.
    remote: Total 3 (delta 1), reused 0 (delta 0)
    Unpacking objects: 100% (3/3), done.
    From /private/tmp/git-subs/main/../remotes/plugin
       0e90143..12e3a52 master -> origin/master
    Submodule path 'vendor/plugins/demo': checked out '12e3a529698c519b2fab790630f71bd531c45727'

I should note there is a **CLI option** that will verify whether currently referenced submodule commits need to be pushed too, and if so will push them: it’s *git push --recurse-submodules=on-demand* (quite a mouthful, admittedly). It needs to have something container-level to push to work, though: only submodules won’t cut it.

What’s more, [EDIT Jan 14, 2016] *(there’s **no configuration setting** for this, so you’d have to standardize procedures around an alias, e.g. spush:) — *starting with Git 2.7.0, there’s now a *push.recurseSubmodules* configurations setting you can define (to *on-demand* or *check*).

    git config --global alias.spush 'push --recurse-submodules=on-demand'

### Removing a submodule

There are two situations where you’d want to “remove” a submodule:

* You **just want to clear the working directory** (perhaps before archiving the container’s WD) but want to retain the possibility of restoring it later (so it has to remain in *.gitmodules* and *.git/modules*);

* You wish to **definitively remove it** from the current branch.

Let’s see each case in turn.

### Temporarily removing a submodule

The first situation is easily handled by *git submodule deinit*. See for yourself:

    main (master u=) $ git submodule deinit vendor/plugins/demo
    Cleared directory 'vendor/plugins/demo'
    Submodule 'vendor/plugins/demo' (../remotes/plugin) unregistered for path 'vendor/plugins/demo'
    main (master u=) $

This has no impact on container status whatsoever. The submodule is not locally known anymore (it’s gone from *.git/config*), so its absence from the working directory goes unnoticed. We still have the *vendor/plugins/demo* directory but it’s empty; we could strip it with no consequence.

The submodule must not have any local modifications when you do this, otherwise you’d need to *--force* the call.

Any later subcommand of *git submodule* will **blissfully ignore this submodule** until you *init* it again, as the submodule won’t even be in local config. Such commands include *update*, *foreach* and *sync*.

On the other hand, **the submodule remains defined** in *.gitmodules*: an *init* followed by an *update* (or a single *update --init*) will restore it as new:

    main (master u=) $ git submodule update --init
    Submodule 'vendor/plugins/demo' (../remotes/plugin) registered for path 'vendor/plugins/demo'
    Submodule path 'vendor/plugins/demo': checked out '12e3a529698c519b2fab790630f71bd531c45727'
    main (master u=) $

### Permanently removing a submodule

This means you want to get rid of the submodule for good: a regular *git rm* will do, just like for any other part of the working directory. This will only work if your submodule uses a *gitfile* (a *.git* which is a file, not a directory), which is the case starting with Git 1.7.8. Otherwise you’ll have to handle this by hand (I’ll tell you how at the end).

In addition to stripping the submodule from the working directory, the command will **update the *.gitmodules* file** so it does not reference the submodule anymore. Here you go:

    main (master u=) $ git rm vendor/plugins/demo
    rm 'vendor/plugins/demo'
    main (master + u=) $ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Changes to be committed:
      (use "git reset HEAD <file>..." to unstage)

    **  modified: .gitmodules
      deleted: vendor/plugins/demo**

    fatal: Not a git repository: 'vendor/plugins/demo/.git'
    Submodule changes to be committed:

    * vendor/plugins/demo 12e3a52...0000000:

Naturally, advanced status info trip over themselves here, because the *gitfile* for the submodule is gone (actually, the entire *demo* directory vanished).

    main (master + u=) $ git ci -m "Removed the demo submodule"
    [master 31cb27d] Removed the demo submodule
     2 files changed, 4 deletions(-)
     delete mode 160000 vendor/plugins/demo

    main (master u+1) $ git status
    On branch master
    Your branch is ahead of 'origin/master' by 1 commit.
      (use "git push" to publish your local commits)
    nothing to commit, working directory clean

What’s odd though, is that the local config retains submodule information, unlike what happens when you *deinit*. So, for a comprehensive removal, I recommend you do both, in sequence, so as to end up properly cleaned up (it wouldn’t work after our previous command, because it cleared *.gitmodules* already):

    git submodule deinit path/to/module *# ensure local config cleanup
    *git rm path/to/module               *# clean WD and .gitmodules*

Regardless of your approach, the submodule’s repo remains present in *.git/modules/vendor/plugins/demo*, but you’re free to kill that whenever you want.

If you ever need to remove a submodule that was set up prior to Git 1.7.8, and therefore embeds its *.git* directory straight in the container’s working directory (instead of relying on a *gitfile*), you’ll have to break out the bulldozer: the previous two commands need to be preceded by a manual folder removal, e.g. *rm -fr vendor/plugins/demo*, because said commands will always refuse to delete an actual repository.

## Best practice recap (TL;DR)

### Configuration settings

* *diff.submodule = log* (so you get clearer container diffs when referenced submodule commits changed).

* *fetch.recurseSubmodules = on-demand* (so you are confident new referenced commits for known submodules get fetched with container updates).

* *status.submoduleSummary = true* (so *git status* gets useful again when a referenced submodule commit changed).

### Adding or cloning

* Initial add: *git submodule add <url> <path>*

* Initial container clone: *git clone --recursive <url> [<path>]*

### Grabbing updates inside a submodule

1. *cd path/to/module*

1. *git fetch*

1. *git checkout -q <commit-sha1>*

1. *cd -*

1. *git commit -am “Updated submodule X to: blah blah”*

### Grabbing container updates

1. *git pull*

1. *git submodule sync --recursive*

1. *git submodule update --init --recursive*

### Updating a submodule inside container code

1. *git submodule update --remote --rebase -- path/to/module*

1. *cd path/to/module*

1. Local work, testing, eventually staging

1. *git commit -am “Update to central submodule: blah blah”*

1. *git push*

1. *cd -*

1. *git commit -am “Updated submodule X to: blah blah”*

### Permanently removing a submodule (1.7.8+)

1. *git submodule deinit path/to/module*

1. *git rm path/to/module*

1. *git commit -am “Removed submodule X”*

## Leftovers

A few submodule-related tidbits are left to mention. Here goes.

### Git commands

* *git submodule foreach* lets you run arbitrary commands on all known (initialized) submodules, recursively or not; commands have access to various environment variables that state the submodule’s path, its referenced commit and the container working directory’s root path. Useful for custom scripting.

* *git submodule status* is a specific status display for submodules, recursive on request. It tells us what the referenced commits are, whether working directories stray from that, whether submodules are initialized yet or not, and even merge conflicts, if any. Faster than manually checking through your working directories.

* *git submodule summary* lists history ranges between the latest referenced commits and the ones currently checked out. This is what *git status* and *git log* display when submodule logs are enabled.

* *git mv* on a 1.7.8+ submodule directory (one with a *gitfile*) does the right thing: it changes the relative path inside the *gitfile*, updates the *core.worktree* reference in the submodule’s repo inside *.git/modules*, and updates and stages *.gitmodules*.

### CLI options

* *git diff --ignore-submodules*, just like *git status --ignore-submodules*, remove any submodule-related information. Country-productive IMHO.

### Configuration settings

* *diff.ignoreSubmodules* permanently zaps out submodule info from diffs. A rather bad idea, if you ask me.

## Want to learn more?

I wrote [a number of Git articles](https://medium.com/@porteneuve), and you might be particularly interested in the following ones:

* [Our GitHub video series is out!](https://medium.com/@porteneuve/our-github-video-course-series-is-out-1fe829e04a59) (absolutely kick-ass, even for experts)

* [Mastering Git subtrees](https://medium.com/p/mastering-git-subtrees-943d29a798ec) (because submodules, I mean, yuck!)

* [Fix conflicts only once with git rerere](https://medium.com/@porteneuve/fix-conflicts-only-once-with-git-rerere-7d116b2cec67) (why do it twice?)

* [How to make Git preserve specific files while merging](https://medium.com/@porteneuve/how-to-make-git-preserve-specific-files-while-merging-18c92343826b) (sweet trick!)

* [Getting solid at Git merge vs. rebase](https://medium.com/@porteneuve/getting-solid-at-git-rebase-vs-merge-4fa1a48c53aa) (a must-read!)

Also, if you enjoyed this post, say so: [upvote it on HN](https://news.ycombinator.com/item?id=9079447)! Thanks a bunch!

Although we don’t publicize it much for now, we do offer English-language Git training across Europe, based on our battle-tested, celebrated [Total Git](http://www.git-attitude.fr/total-git/) training course. If you fancy one, just [let us know](http://www.git-attitude.fr/request-intra-or-custom)!

*(We can absolutely come over to US/Canada or anywhere else in the world, but considering you’ll incur our travelling costs, despite us being super-reasonably priced, it’s likely you’ll find a more cost-effective deal using a closer provider, be it GitHub or someone else. Still, if you want **us**, follow the link above and let’s talk!)*
