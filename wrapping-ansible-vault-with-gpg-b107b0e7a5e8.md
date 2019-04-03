
# Wrapping Ansible Vault with gpg

Ansible Vault is kind of limited for my usual experience because it requires you to type in a password.

Yeah, it’s a bit of a silly complaint. But I really would like to type as few passwords as possible; vault doesn’t do caching.

Vault also offers the option to have a password file where you can read the password from. That’s stupid. But it does allow you to just put an executable there, and then uses the return value of the executable as the password. *Goooood.*

So some quick config manipulation:

<iframe src="https://medium.com/media/ac99e4745b48d8c63394542b0ff32ac2" frameborder=0></iframe>

Voila! Just use Vault to drop in the passwords either inline (starting with 2.3+) or in Vault files like $site/group_vars/$group/my-auth; store the encryption password for the Vault in a GPG-encrypted file and set that as VAULT_PW_FILENAME in the script. Your GPG agent now handles credential caching for Ansible.

Bonus: just add your coworkers’ keys if you have multiple collaborators on something (or are just archiving the customer configuration for backup). Even allows crude ACLs if you split up the group variables fine enough.
