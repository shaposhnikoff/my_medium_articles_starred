
# Create a custom module with Ansible-Python

Ansible modules

Ansible ships with a number of modules (called the ‘module library’) that can be executed directly on remote hosts or through Playbooks.

Users can also write their own modules. These modules can control system resources, like services, packages, or files (anything really), or handle executing system commands.

![](https://cdn-images-1.medium.com/max/2000/1*ISR7ykoXb5VyESCWdbkZBQ.jpeg)

**Why create your own module?**

As said earlier, ansible has lot of modules to perform various tasks. But different environment can have different requirements wherein you would prefer to have a custom module to handle the system or environment or project in your own way.

Lets understand how we can create a custom module in an easy way.

**Folder structure**

It is important to know the folder and file structure before we begin to create a custom module. Check below image for reference. What we need to work on is main.yaml and testing.py under library folder.

root@ubuntu:~/ANSIBLEtesting# tree
.
└── playbooks
 ├── library
 │ └── testing.py
 └── main.yaml

2 directories, 2 files

**Write your first python program**

To write ansible pragmatically we must understand module utilities. Ansible provides a number of module utilities that provide helper functions that you can use when developing your own modules. The testing.py module utility provides the main entry point for accessing the Ansible library, and all Ansible modules must, at minimum, import from testing.py.

    from ansible.module_utils.basic import *

This will import all the module_utils files to create a custom module. The output format of the python file is only JSON. So when you print the output of a python file, it will have to be in JSON.

Key parts include always importing the boilerplate code from ansible.module_utils.basic like this:

    from ansible.module_utils.basic import AnsibleModule
    if __name__ == '__main__':
        main()

Now instantiate the main class or you can write directly to the main class.

    if __name__ == '__main__':
      fields = {
      "yourName": {"required": True, "type": "str"}
      }
      module = AnsibleModule(argument_spec=fields)
      yourName = os.path.expanduser(module.params['yourName'])
      newName = firstProg(yourName)
      module.exit_json(msg=newName)

The AnsibleModule provides lots of common code for handling returns, parses your arguments for you, and allows you to check inputs. Here yourName is the input given to the python file. This input will be taken from ansible playbook.

Successful returns are made like this:

    module.exit_json(changed=True, something_else=12345)

And failures are just as simple (where msg is a required parameter to explain the error):

    module.fail_json(msg="Something fatal happened")

The final python program looks like this

    #!/bin/env python
    from ansible.module_utils.basic import *
    import os, json
    import re, sys

    def firstProg(text):
      text1 = "Hello " + text
      return text1

    if __name__ == '__main__':
      fields = {
      "yourName": {"required": True, "type": "str"}
      }
      module = AnsibleModule(argument_spec=fields)
      yourName = os.path.expanduser(module.params['yourName'])
      newName = firstProg(yourName)
      module.exit_json(msg=newName)

**Get your playbook ready**

Know that we will use the name of the custom module same as the python file name.

    - hosts: all
      remote_user: root
      gather_facts: yes

      vars_prompt:
       - name: giveName
         prompt: "Please provide your name"
         private: no
         failed_when: giveName is undefined

     tasks:
      - name: Python Execution
        testing: yourName={{ giveName }}
        register: result
      - debug: var=result

Here vars_prompt will ask you for a name and testing module under task will pass that name to the python script. Can you guess what the output will be ? Lets check.

    ansible-playbook main.yaml

**Conclusions**

You can check more in detail about the custom modules in[ Ansible Documentation](http://docs.ansible.com/ansible/latest/dev_guide/developing_modules.html). There is much more that can be done with custom modules.

This article has been taken directly from [www.9tocloud.com](http://www.9tocoud.com)
