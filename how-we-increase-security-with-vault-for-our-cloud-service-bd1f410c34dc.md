
# How we increase security with Vault for our Cloud service

How we increase security with Vault for our Cloud service

![](https://cdn-images-1.medium.com/max/2000/0*_HCtm4fvN3cETFaz.png)

Ansible Vault, Hashicorp Vault? These passwords managers are confusing. They have a similar names. What’s the difference between them? When you provide a Cloud instance, as we do with [Mytuleap.com](https://www.mytuleap.com/) -the name of the hosted platform of [Tuleap Enterprise](https://www.enalean.com/enterprise-solutions)-, it with comes with a lot of credentials like Mysql passwords, Libnss and your admin connection password. All these information are of course considered as critical. To ensure a security without fail, we had try out many solutions. Vault has proved to be the best of them. Explanations.

We added a new physical server linked to our Tuleap infrastructure to retrieve, stock and encrypt all platforms passwords :

## Hashicorp Vault

Vault secret manager is a new tool developed by Hashicorp who also made Vagrant, Terraform and Consul. This tool uses high level encryption AES 256 and control access by tokens, credentials, certificates and/or API keys.

Vault can store infrastructure secrets, or dynamically generate token to third-party ressources with ttl (time to live) durations. Datas are then encrypted in its backend. Vault is also completely free and open source.

## Fine grain access control

Vault provides custom policies that can be associated with a generated token to manage access allowing a strict control by the administrator.

## Key Sharing

![](https://cdn-images-1.medium.com/max/2000/0*e6NGv8d8HYJxcf9n.png)

At first with, the vault is in a sealed state. When you want to initialize it, the vault creates a master key (Encryption Key) then it uses the Shamir’s secret sharing algorithm to split the masters into shares. You choose how many keys you want to split the master and how many keys are required to unseal the vault. Required keys are called « The Threshold ».

Command :

    vault init -key-shares=5 -key-threshold=3

## We apply good practice

The Threshold must be dispatched between trustworthy persons. Good practice teaches us to let the vault unseald as long as there is no problem detected. In our case Threshold’s keys can also be encrypted by PGP keys to maximize the security protection

## Key rotation

The rekey operation is used to generate a new master key. It allows to change the parameters of the key splitting, so that the number of shares and the threshold required to unseal can be changed. To perform a rekey a threshold of the current unseal keys must be provided. This is to prevent a single malicious operator from performing a rekey and invalidating the existing master key.

Command via API :

    curl --header "X-Vault-Token: ..." --request DELETE [https://vault.rocks/v1/sys/rekey/init](https://vault.rocks/v1/sys/rekey/init)

## Root Token

Vault also provides a root token to manage all policies linked to children tokens. Root tokens are tokens that have the root policy attached to them. Root tokens can do anything in Vault. They are the only type of token within Vault that can be set to never expire without any renewal needed.

Root tokens should be extremely carefully guarded in production. In fact, the Vault team recommends that root tokens are only used for the initial setup (usually, setting up authentication backends and policies necessary to allow administrators to acquire more limited tokens) or in emergencies, and are revoked immediately after they are no longer needed. If a new root token is needed, the generate-root command and associated API endpoint can be used to generate one.

## Our strategy to unseal the Vault

![](https://cdn-images-1.medium.com/max/2000/0*yXr-GoXqDW17BYCv.png)

Every reliable person who has an unseal key must encrypt this key with his public PGP key make sure that if he losts the key, nobody will be able to use it. So to unseal the vault everybody has to decrypt his key then send it to vault.

Command :

    Vault unseal add key1 add key2 add key3 ...

Then the sealed status will change to be False.

## Ansible Vault

In a previous article, we explained our Tuleap cluster made with this docker swarm. We used ansible to automate our apps and IT infrastructure deployment, in association with vault to secure clear tokens.

Vault is an Ansible’s feature that came in version 1.5 that allows keeping sensitive data such as credentials, passwords and keys encrypted rather than as plaintext in your playbooks or roles.

The vault feature can encrypt any structured data file used by Ansible. This can include “group_vars” , “host_vars” or ”vars_files”.

In our case, we stock Hashicorp vault token with specific policy in ours ansible files ”vars_files”, with this ansible vault we can encrypt this file in AES256 algorithm.

Creating Encrypted Files :

    ansible-vault create foo.yml

Ansible will ask you to create a new password. The password used with the vault currently must be the same for all the files you wish to use at the same time.

After providing a password, the tool will launch whatever editor you have defined with $EDITOR, and defaults to vi. Once you are done with the editor session, the file will be saved as encrypted data.

Editing Encrypted Files

    ansible-vault edit foo.yml

Rekeying Encrypted Files

    ansible-vault rekey foo.yml

## Using PGP to encrypt the Ansible Vault password

Ansible-vault can be a sort of pain. Every time you want to edit an encrypted file or running ansible playbook you have to enter your password. If you are doing plenty of test it can be tiring to enter it again and again.

There is a solution to avoid this. Ansible has support for getting the vault passphrase from a script.

## Here are tips

1. First, write your password in a text file then use your PGP key to encrypt your password’s file. Now you have an encrypted password, you can delete your clear text password. (extract from [https://blog.erincall.com/p/using-pgp-to-encrypt-the-ansible-vault](https://blog.erincall.com/p/using-pgp-to-encrypt-the-ansible-vault))

1. Now you can make a light script to read your encrypted file.

    #!/bin/sh gpg --batch --use-agent --decrypt vault_passphrase.gpg

3. To finish you have to tell ansible how to read this password. In the ansible configuration file you can add this line :

    [defaults] vault_password_file=open_the_vault.sh

With this solution you no longer need to enter your password manually.

## To put it in a nutshell

![](https://cdn-images-1.medium.com/max/2000/0*vtdtomQTROn9MzWf.png)

* We dispatch and encrypt Hashicorp-Vault unseal keys with each administrator public PGP keys

* We use ansible deployment to retrieve all tuleap passwords in our cluster

* To communicate with Hashicorp Vault server we create a token linked to a specific policy

* This token is also encrypted with ansible-vault and protected by a password

* Ansible-vault password is encrypted with PGP keys*.*
