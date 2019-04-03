
# About Ansible Vault



New in Ansible 1.5, “Vault” is a feature of ansible that allows keeping sensitive data such as passwords or keys in encrypted files, rather than as plain text in your playbooks or roles. These vault files can then be distributed or placed in source control. To enable this feature, a command line tool, ansible-vault is used to edit files, and a command line flag –ask-vault-pass or –vault-password-file is used. Alternately, you may specify the location of a password file or command Ansible to always prompt for the password in your ansible.cfg file. These options require no command line flag usage.

## How to encrypt the Ansible Playbook

Use the option “encrypt” along with the ansible-vault command. Enter the vault password twice you wish to set for the particular playbook users.yml, this password is only for this file.

    ansible-vault encrypt users.yml
    New Vault password:
    Confirm New Vault password:
    Encryption successful

Yes, “users.yml” is encrypted.

Now, if anyone tries to open the protected file with any normal editors, they cannot be readable by the users. because it's encrypted.

    cat users.yml
    $ANSIBLE_VAULT;1.1;AES256
    32616565646435323531613532376266653831663865613237626534636231366238386361303436
    6265333530343864336132376338356666646433656232320a636633663965373366613561343634
    31343536393265646661643764356666336536616461663263333161346537633031326138353864
    6334663933303634350a666337636430653435396265356138383839353434376432326131303131
    3362

## How to view the encrypted playbook file?

Use the “view” option along with ansible-vault command and enter the vault password.

    ansible-vault view users.yml
    Vault password:
    ---
    - hosts: clients
      tasks:
      - name: Adding Users
        user:
         name: anmol
         password: devops@123
         comment: "Anmol Nagpal"
         shell: /bin/bash
         createhome: yes
         home: /home/anmol

## How to edit the encrypted playbook file?

Use the “edit” option along with ansible-vault command and enter the vault password. This will use your default editor set in your user environment.

    ansible-vault edit users.yml
    Vault password:

## How to run an encrypted ansible playbook file?

If a playbook is encrypted, We cannot run an ansible-playbook as we do normally. Else you would get an error as below.

    ansible-playbook users.yml
    ERROR! Attempting to decrypt but no vault secrets found

Instead, we can use use the argument “ — ask-vault-pass” to provide the vault password or Save your vault password in a file and call the vault password file using the argument “ — vault-password-file”.

## Using the argument “ — ask-vault-pass”

    ansible-playbook users.yml --ask-vault-pass
    Vault password:

## Using the argument “ — vault-password-file”

    ansible-playbook users.yml --vault-password-file /anmol/.ansible/vault-passwd

## How to change the existing vault password?

Use the “rekey” option along with the ansible-vault command. Enter the old vault password and enter the new password twice.

    ansible-vault rekey users.yml
    Vault password:
    New Vault password:
    Confirm New Vault password:
    Rekey successful

## How to decrypt the protected ansible playbook file?

Use the “decrypt” option along with the ansible-vault command,

    ansible-vault decrypt users.yml
    Vault password:
    Decryption successful
