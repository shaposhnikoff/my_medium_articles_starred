
# Use cases of Ansible Vault

We all know, in general ‘vault’ means a compartment or room for safekeeping of valuables.

Like that in ansible we are using a command line tool called ‘ansible-vault’ to safely keep the variables or passwords or files in an encrypted format. So it is an encrypted store.

It uses Advanced Encryption Standard AES-256 with a password as the secret key.

**Usage help for ansible-vault:**

To know how to use the ‘ansible-vault’, type the below command.

*[root@centos7 ~]# ansible-vault — help*
 *Usage: ansible-vault [create|decrypt|edit|encrypt|encrypt_string|rekey|view] [options] [vaultfile.yml]*

*encryption/decryption utility for Ansible data files*

*Options:*
 *— ask-vault-pass ask for vault password*
 *-h, — help show this help message and exit*
 *— new-vault-id=NEW_VAULT_ID*
 *the new vault identity to use for rekey*
 *— new-vault-password-file=NEW_VAULT_PASSWORD_FILES*
 *new vault password file for rekey*
 *— vault-id=VAULT_IDS the vault identity to use*
 *— vault-password-file=VAULT_PASSWORD_FILES*
 *vault password file*
 *-v, — verbose verbose mode (-vvv for more, -vvvv to enable*
 *connection debugging)*
 *— version show program’s version number and exit*

*See ‘ansible-vault <command> — help’ for more information on a specific*
 *command.*
 *[root@centos7 ~]#*

Lets see the various use cases of anisble-vault as below,

**1) To encrypt a file via ansible-vault:**

# ansible-vault create <filename>

Lets create an inventory file named ‘staticinventory.txt’ with below contents under ‘/root/inventory’ directory,

*centos6web*
 *centos6db*
 *centos7*

*[root@centos7 inventory]# ansible-vault create staticinventory.txt*
 *New Vault password:*
 *Confirm New Vault password:*

After entering the password, it will open a file in ‘vi’ editor with insert mode. We can type the contents and save it.

Now encrypted ‘staticinventory.txt’ file has been created.

**2)To read an already encrypted file via ansible-vault:**

In the previous step we have encrypted a file, now try to read with normal ‘cat’ command and see,

*[root@centos7 inventory]# pwd*
 */root/inventory*
 *[root@centos7 inventory]# ls*
 *staticinventory.txt*
 *[root@centos7 inventory]# cat staticinventory.txt*
 *$ANSIBLE_VAULT;1.1;AES256*
 *61663732633635363162303135396462643430633263316633313233356264303066383735386637*
 *3732666539383866643232306439303166636632656333360a316332386662653836653465326333*
 *66636634633433636466363832623665326539313431303165313137653530323262633131613632*
 *3962643633646630640a656534303663656130323036613336313139383134336234353032343635*
 *30656434376330333066353531646565663631643832663531353133376435643836*
 *[root@centos7 inventory]#*

Lets view it using ansible-vault

*[root@centos7 inventory]# ansible-vault view staticinventory.txt*
 *Vault password:*
 *centos6web*
 *centos6db*
 *centos7*
 *[root@centos7 inventory]#*

**3) To write ie. edting an already encrypted file via ansible-vault:**

# ansible-vault edit <filename>

*[root@centos7 inventory]# ansible-vault edit staticinventory.txt*
 *Vault password:*

Please enter the password and proceed to make changes in the file.

**4)To change the password of an already encrypted file**

#ansible-vault rekey <filename>

*[root@centos7 inventory]# ansible-vault rekey staticinventory.txt*
 *Vault password:*
 *New Vault password:*
 *Confirm New Vault password:*
 *Rekey successful*
 *[root@centos7 inventory]#*

**5) To decrypt an existing file via ansible-vault:**

#ansible-vault decrypt <filename>

*[root@centos7 inventory]# ansible-vault decrypt staticinventory.txt*
 *Vault password:*
 *Decryption successful*
 *[root@centos7 inventory]#*

Now existing encrypted file ‘staticinventory.txt’ has been decrypted with above command. Now we can use that file without ansible-vault as below,

*[root@centos7 inventory]# cat staticinventory.txt*
 *centos6web*
 *centos6db*
 *centos7*
 *[root@centos7 inventory]#*

In this article we have seen 5 different use ceases of ansible-vault.
