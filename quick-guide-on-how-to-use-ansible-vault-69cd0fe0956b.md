
# Quick guide on how to use Ansible Vault

Create secured password storage (Ansible Vault) to access servers

    ansible-vault create vault/my.vault

It will ask you for a password for this vault file. Create one and remember it!

2. Put the following in the file:

    vault_ansible_user: SERVER_USERNAME
    vault_ansible_password: SERVER_PASSWORD
    vault_ansible_become_pass: SERVER_PASSWORD

3. Create file “~/.vault_pass” and put there your Vault password you created in step 1.

4. Run your playbook file:

    ansible-playbook   \
        -i inventories/dit/hosts.ini site.yml \
        --extra-vars [@vaults/m](http://twitter.com/vaults/akakhan)y.vault \
        --vault-id ~/.vault_pass

To run a single command you can use:

    ansible my_servers --extra-vars [@vaults/m](http://twitter.com/vaults/akakhan)y.vault --vault-id ~/.vault_pass -i inventories/dit/hosts.ini -m ping
