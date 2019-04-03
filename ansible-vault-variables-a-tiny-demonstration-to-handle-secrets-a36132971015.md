
# Ansible Vault Variables — a Tiny Demonstration to Handle Secrets

How can we avoid plain text values to store critical values like passwords in an Ansible Play. Here is a tiny demonstration.

![](https://cdn-images-1.medium.com/max/2000/1*oz1Lof-yzug0HiuH7fsa7w.png)

First let us ready the variable by encrypting it. You enter the ansible-vault command in the Ansible control machine (the machine that has Ansible installed)

    **ansible-vault encrypt_string**
    New Vault password: *<---- you enter your password*
    Confirm New Vault password: *<---- you re-enter the password*
    Reading plaintext input from stdin. (ctrl-d to end input)
    **Hello World ***<---- This is our confidential data*
    !vault |
              $ANSIBLE_VAULT;1.1;AES256
              63353565323632376235366164613530666536653063323762363833376637386262363737386636
              3531393131633962336161356561666561366238356162310a653637393963636136393464306635
              37363964633832313833346538383262653635653930346263336538326438633764313936666533
              3564643232333065310a353735333737383832333033336665633165623161343736353438386430
              6466
    Encryption successful

Copy the above encrypted data and paste it in the Ansible play as below.

    - name: Vault Demo
      hosts: localhost
      gather_facts: false
      connection: local
      vars:
        notsecret: Hello123
        mysecret: !vault |
                $ANSIBLE_VAULT;1.1;AES256
                63353565323632376235366164613530666536653063323762363833376637386262363737386636
                3531393131633962336161356561666561366238356162310a653637393963636136393464306635
                37363964633832313833346538383262653635653930346263336538326438633764313936666533
                3564643232333065310a353735333737383832333033336665633165623161343736353438386430
                6466
    
      tasks:
        - name: Public 
          debug: msg="Public info..{{ notsecret }}"

    - name: Secret
          debug: msg="Secret info.. {{ mysecret }}"
    

When we run the above play we need to provide the extra option like this.

    **ansible-playbook play.yml --ask-vault-pass**

or have a file containing your vault password and pass it as below. This will not prompt for the vault password while running.

    **ansible-playbook play.yml  --vault-password-file ./play-vault-pass.txt**

Remember to not to expose the password file in case this option is used!

Thanks for your time, please do follow for more such tiny demo snippets!
