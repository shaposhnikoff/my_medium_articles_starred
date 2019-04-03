
# You should learn ansible, right now

(or anything that moves like ansible)

![](https://cdn-images-1.medium.com/max/2522/1*2f9LDb-D1xjKQBwI4dCFmQ.png)

Since I graduated from college I am working as a one-man-show for a lot of projects mainly. In 2012 it just was called “server stuff” and I did it for all my projects on my own. I rented the first unmanaged vserver right away and got it rolling. Since then devops never left my every day working life.

When my wife and I started [our own SaaS](https://www.ich-will-ein-pony.de/de/keepsake-journal) I had to improve drastically to match the new challenges we faced. I used ansible before, got the concept and used it here and there, but never got a real connection with it. I often forget the yaml-structure for the most common things: “do I indent this, or that? Where does this parameter belong to?” The module documentation needs some time getting used to it and I did not like the logging at all.

Lately I moved from atlassian bamboo to jetbrains teamcity and my whole build system for all custom projects need to be moved to the new CI-Server. I had tons of custom task scripts all tightly bound to environment variables from bamboo, which would be provided from the build server or the agent. They we’re nearly useless for the migration. Let alone the tasks that only configured plugins from bamboo.

After evaluation other options like ansible, the conclusion was drawn very fast: right now ansible seems to me the best solution as a server management tool for distributed servers and a replacement for a lot of bash scripting. So I started to read bits of the documentation for ansible again — This time with the upcoming task in mind, to migrate all continuous integration scripts to ansible.
After a few days I started converting the first scripts into a reusable playbook, that would provide tasks to build all my projects in teamcity.

It was a blast!

Not only I understood the yaml-structures much better, I remembered parameters from a lot of the common modules like file, sync, template much better and I found neat tricks in jinja2 — which is fortunately very similar to twig. Once I understand that some are module parameters but every module has generic parameters as well (when, vars, loops) the yaml-structure was so easy to understand. I customized the ansible daemon to have a better logging output written in yaml, a breeze to debug.
> I proved my own point once again: If something you have to use something on a daily basis as a tool, but it seems not to be so much fun: put some effort in it, to become better with it and use your tool with more fun and more effect.

Learn to use a drilling machine, if you’re planning to tight a lot of screws

Ansible solved my aching for more security on ci servers with secrets and private ssh-keys. The private data can be encrypted with [ansible vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) and then decrypted on the continuous integration server. It then makes it easy to store the secrets in youre vcs without compromising security.

I learned about [lookups](https://docs.ansible.com/ansible/latest/plugins/lookup.html#plugin-list) which allowed me to use all env variables, coming from the ci or from my developer shell and convert them to usable variables in my playbook. This is so much better than passing all variables with --extra-varsaround.

The best feature I learned and reused a lot, was the [include_tasks](https://docs.ansible.com/ansible/2.4/include_tasks_module.html) module. Its so simple, but it allows you to create a dynamic playbook in your project, that uses pre-defined tasks in a base-playbook that is provided by your ci-server or a separate vcs-repository with build scripts / build playbooks. I never though of it before. It’s like composing a lot of bash scripts, but so much simpler and cleaner

    - name: build local iwep-www package
      hosts: 127.0.0.1
      connection: local

    gather_facts: false

    roles:
        - build-from-teamcity
        - docker-compose-build

    tasks:
        - name: "load variables for env"
          include_vars: "{{ src }}/etc/symfony/parameters.dev.yml"

    - name: build
          block:
            - include_tasks: '{{ compose_tasks }}/with-registry.yml'

            # write into src directory to build
            - include_tasks: '{{ compose_tasks }}/write-env.yml'
              vars:
                dest: "{{ src }}"

            - include_tasks: '{{ compose_tasks }}/compose-build-ssh.yml'

            - include_tasks: '{{ compose_tasks }}/compose-push.yml'

            - include_tasks: '{{ compose_tasks }}/create-package.yml'

    always:
            - include_tasks: '{{ compose_tasks }}/always.yml'

That is one clean build script, and it lives in the repository of the project. So everyone developing on this project is able to modify the build on the ci-server without having to look or login.
If one task would not fit perfectly, just replace some variables in it, or copy the whole included tasks an modify it to your needs. If you do this to often, refactor for a new included-task in your build-playbook, so leverage the refectoring in other projects.

## My recommendation

Go ahead and learn something like ansible, right now because some day it’ll be too late. It’s good to know a lot of bash, but something like ansible, easily usable as an every day tool and executes on remotes, will open a whole new world of automatisation for you.

Automate your ci, automate your devops-tasks, automate healthchecks, automate deployments.

Ansible is worth the effort to get to know it and properly use it. I wish I would started earlier with ansible and already have a pile of nice, structured ansible-playbooks to use for every little bit of devops. I’m still converting old bash scripts to ansible and it’ll take me a while to automate everything.

## Did this article help you?

Plase clap it a few or more times, so others can find it and it’ll help them, too. Thanks!
