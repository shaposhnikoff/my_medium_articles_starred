
# Protecting Your Secrets with Ansible

Every business is a digital business. Technology is your innovation engine, and delivering your applications faster helps you win. Historically, that required a lot of manual effort and complicated coordination. But today, there is Ansible — the simple, yet powerful IT automation engine that thousands of companies are using to drive complexity out of their environments and accelerate DevOps initiatives.

[Ansible](http://www.bigdatatraining.in/ansible-training/) is a great tool for handling deployment as well as configuration management, providing a lot more functionality than shell scripts. It doesn’t require that one learn a new set of abstractions to hide the differences between operating systems.

In this Blog, we cover how to keep your secrets safe with [Ansible:](http://www.bigdatatraining.in/ansible-training/)

* Encrypting data at rest

* Protecting secrets while operating

Secrets are meant to stay secret. Whether they are login credentials to a cloud service or passwords to database resources, they are secret for a reason. Should they fall into the wrong hands, they can be used to discover trade secrets, customers’ private data, create infrastructure for nefarious purposes, or worse. All of which could cost you or your organization a lot of time, money, and headache!

## [Encrypting data at rest](http://www.bigdatatraining.in/ansible-training/)

As a configuration management system or an orchestration engine, Ansible has great power. In order to wield that power, it is necessary to entrust secret data to [Ansible](http://www.bigdatatraining.in/ansible-training/). An automation system that prompts the operator for passwords each connection is not very efficient. To maximize the power of [Ansible](http://www.bigdatatraining.in/ansible-training/), secret data has to be written to a file that [Ansible](http://www.bigdatatraining.in/ansible-training/) can read and utilize the data from within.

This creates a risk though! Your secrets are sitting there on your filesystem in plain text. This is a physical and digital risk. Physically, the computer could be taken from you and pawed through for secret data. Digitally, any malicious software that can break the boundaries set upon it could read any data your user account has access to. If you utilize a source control system, the infrastructure that houses the repository is just as much at risk.

Thankfully, [Ansible ](http://www.bigdatatraining.in/ansible-training/)provides a facility to protect your data at rest. That facility is Vault. This facility allows for encrypting text files so that they are stored at rest in an encrypted format. Without the key or a significant amount of computing power, the data is indecipherable.

The key lessons to learn when dealing with encrypting data at rest are:

* Valid encryption targets

* Creating new encrypted files

* Encrypting existing unencrypted files

* Editing encrypted files

* Changing the encryption password on files

* Decrypting encrypted files

* Running ansible-playbook referencing encrypted files

## [Things Vault can encrypt](http://www.bigdatatraining.in/ansible-training/)

The Vault feature can be used to encrypt any **structured data** file used by Ansible. This is essentially any YAML (or JSON) file that Ansible uses during its operation. This can include:

* group_vars/ files

* host_vars/ files

* include_vars targets

* vars_files targets

* — extra-vars targets

* role variables

* role defaults

* task files

* handler files

* source files for copy module

If the file can be expressed in YAML and read by Ansible, or if the file is to be transported with the copy module, it is a valid file to encrypt with Vault. Because the entire file will be unreadable at rest, care should be taken to not be overzealous in picking which files to encrypt. Any source control operations with the files will be done with the encrypted content, making it very difficult to peer review. As a best practice, the smallest amount of data possible should be encrypted, which may even mean moving some variables into a file all by themselves.

## [Creating new encrypted files](http://www.bigdatatraining.in/ansible-training/)

To create new files, Ansible provides a new program, ansible-vault. This program is used to create and interact with Vault encrypted files. The subroutine to create encrypted files is the create subroutine:

![](https://cdn-images-1.medium.com/max/2000/0*5k9hHF6hZZgxDEiE.)

To create a new file, you’ll need to know two things ahead of time. The first is the password Vault should use to encrypt the file, and the second is the file name itself. Once provided with this information, ansible-vault will launch a text editor, whichever editor is defined in the environment variable EDITOR. Once you save the file and exit the editor, ansible-vault will use the supplied password as a key to encrypt the file with AES256 cypher.

The ansible-vault program will prompt for a password, unless the path to a file is provided as an argument. The password file can either be a plain text file with the password stored as a single line, or it can be an executable file that outputs the password as a single line to standard out.

Let’s walk through a few examples of creating encrypted files. First, we’ll create one and be prompted for a password, then we will provide a password file, and lastly we’ll create an executable to deliver the password.

## PASSWORD PROMPT

![](https://cdn-images-1.medium.com/max/2000/0*JmwHjHBynuxuT6pQ.)

Once the passphrase is entered, our editor opens and we’re able to put content into the file:

![](https://cdn-images-1.medium.com/max/2000/0*3Ab1YdPqi4nXQSde.)

Now, we save the file. If we try to read the contents, we’ll see that they are in fact encrypted, with a small header hint for Ansible to use later:

![](https://cdn-images-1.medium.com/max/2000/0*H_sCYh3C4Ckmd9P3.)

## PASSWORD FILE

In order to use ansible-vault with a password file, we first need to create the password file. Simply echoing a password into a file can do this. Then, we can reference this file when calling ansible-vault to create another encrypted file:

![](https://cdn-images-1.medium.com/max/2000/0*nJEBAbGXsTO2F68u.)

## PASSWORD SCRIPT

This last example uses a password script. This is useful for designing a system where a password can be stored in a central system for storing credentials and shared with contributors to the playbook tree. Each contributor could have his or her own password to the shared credentials store, where the Vault password would be retrieved from. Our example will be far more simple: just a simple output to standard out with a password. This file will be saved as password.sh. The file needs to be marked as an executable for Ansible to treat it as such:

![](https://cdn-images-1.medium.com/max/2000/0*bRiEPiANC4It0gdO.)

## [Encrypting existing files](http://www.bigdatatraining.in/ansible-training/)

The previous examples all dealt with creating new encrypted files using the create subroutine. But what if we want to take an established file and encrypt it? A subroutine exists for this as well. It is named encrypt:

![](https://cdn-images-1.medium.com/max/2000/0*0-bgn67lZNCiGAuT.)

As before our editor opens up, with our content in plain text visible to us. As with create, encrypt expects a password (or password file) and the path to a file. In this case, however, the file must already exist. Let’s demonstrate this by encrypting an existing file:

![](https://cdn-images-1.medium.com/max/2000/0*2E_BnqbUP7s3K7rM.)

We can see the file contents before and after the call to encrypt, whereafter the contents are indeed encrypted. Unlike the create subroutine, encrypt can operate on multiple files, making it easy to protect all the important data in one action. Simply list all the files to be encrypted, separated by spaces.

## [Editing encrypted files](http://www.bigdatatraining.in/ansible-training/)

Once a file has been encrypted with ansible-vault, it cannot be directly edited. Opening the file in an editor would result in the encrypted data being shown. Making any changes to the file would damage the file and Ansible would be unable to read the contents correctly. We need a subroutine that will first decrypt the contents of the file, allow us to edit those contents, and then encrypt the new contents, before saving it back to the file. Such a subroutine exists, called edit:

![](https://cdn-images-1.medium.com/max/2000/0*wOM9Q5h52PbMy-1U.)

As previously our editor opens up, with our content in plain text visible to us. All our familiar options are back, an optional password file/script and the file to edit. If we edit the file we just encrypted, we’ll notice that ansible-vault opens our editor with a temporary file as the file path. The editor will save this and then ansible-vault will encrypt it and move it to replace the original file:

![](https://cdn-images-1.medium.com/max/2000/0*0T3zMcOOo42L20Ep.)

![](https://cdn-images-1.medium.com/max/2000/0*aZ7b2RUyqmQ0rewo.)

## [Password rotation on encrypted files](http://www.bigdatatraining.in/ansible-training/)

Over time, as contributors come and go, it is a good idea to rotate the password used to encrypt your secrets. Encryption is only as good as the protection of the password. ansible-vault provides a subroutine that allows us to change the password, named rekey:

![](https://cdn-images-1.medium.com/max/2000/0*wqvQ_ibvNVxvoGmv.)

The rekey subroutine operates much like the edit subroutine. It takes in an optional password file/script and one or more files to rekey. Note that, while you can supply a file/script for decryption of the existing files, you cannot supply one for the new passphrase. You will be prompted to input the new passphrase. Let’s rekey our even_more_secrets.yaml file:

![](https://cdn-images-1.medium.com/max/2000/0*5W8-gTO2yCsZIwNX.)

## [Decrypting encrypted files](http://www.bigdatatraining.in/ansible-training/)

If, at some point, the need to encrypt data files goes away, ansible-vault provides a subroutine that can be used to remove encryption for one or more encrypted files. This subroutine is (surprisingly) named decrypt:

![](https://cdn-images-1.medium.com/max/2000/0*2-1fyTfb8KIhzbSf.)

Once again, we have an optional argument for a password file/script and then one or more file paths to decrypt. Let’s decrypt the file we created earlier, using our password file:

![](https://cdn-images-1.medium.com/max/2000/0*ZLkXF5PIHhaMbV4s.)

## [Executing Ansible-playbook with encrypted files](http://www.bigdatatraining.in/ansible-training/)

Let’s create a simple playbook named show_me.yaml that will print out the value of the variable inside of a_vars_file.yaml, which we encrypted in a previous example:

    - name: show me an encrypted var 
     hosts: localhost 
     gather_facts: false 
     
     vars_files: 
     — a_vars_file.yaml 
     
     tasks: 
     — name: print the variable 
     debug: 
     var: something

![](https://cdn-images-1.medium.com/max/2000/0*bb3z9qfUvQBS1Tjt.)

## [Protecting secrets while operating](http://www.bigdatatraining.in/ansible-training/)

### Secrets transmitted to remote hosts

Ansible will combine module code and arguments and write this out to a temporary directory on the remote host. This means your secret data is transferred over the wire **AND** written to the remote filesystem. Unless you are using a connection plugin other than ssh, the data over the wire is already encrypted preventing your secrets from being discovered by simple snooping. If you are using a connection plugin other than ssh, be aware of whether or not data is encrypted, while in transit. Using any connection method that is not encrypted is strongly discouraged.

Once the data is transmitted, Ansible may write this data out in clear form to the filesystem. This can happen if pipelining is not in use, **OR** if Ansible has been instructed to leave remote files in place via the ANSIBLE_KEEP_REMOTE_FILES environment variable. Without pipelining, Ansible will write out the module, code plus arguments, into a temporary directory that is to be deleted upon execution. Should there be a loss of connectivity between writing out the file and executing it, the file will be left on the remote filesystem until manually removed.

If Ansible is explicitly instructed to keep remote files in place, then, even if pipelining is enabled, Ansible will write out and leave a remote file in place. Care should be taken with these options when dealing with highly sensitive secrets, even though, typically, only the user Ansible logs in as on the remote host should have access to the leftover file. Simply deleting anything in the ~/.ansible/tmp/ path for the remote user will suffice to clean secrets.

### [Secrets logged to remote or local files](http://www.bigdatatraining.in/ansible-training/)

When Ansible operates on a host, it will attempt to log the action to syslog. If this action is being done with a user with appropriate rights, it will cause a message to appear in the syslog file of the host. This message includes the module name and the arguments passed along to that command, which could include your secrets. To prevent this from happening, a play and task key exists named no_log. Setting no_log to true will prevent Ansible from logging the action to syslog.

Let’s take our previous example of displaying an encrypted secret and add a no_log key to the task to prevent showing its value:

     — — 
    - name: show me an encrypted var 
     hosts: localhost 
     gather_facts: false 
     
     vars_files: 
     — a_vars_file.yaml 
     
     tasks: 
     — name: print the variable 
     debug: 
     var: something 
     no_log: true

If we execute this playbook, we should see that our secret data is protected:

![](https://cdn-images-1.medium.com/max/2000/0*Jq5NpxJVofw4Eskc.)

## [Conclusion](http://www.bigdatatraining.in/ansible-training/)

Ansible can deal with sensitive data. It is important to understand how this data is stored at rest and how this data is treated when utilized. With a little care and attention, Ansible can keep your secrets secret. Encrypting secrets with ansible-vault can protect them while dormant on your filesystem or in a shared source control repository. Preventing Ansible from logging task data can protect against leaking data to remote log files or onscreen displays.
