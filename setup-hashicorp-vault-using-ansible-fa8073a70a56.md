
# Setup Hashicorp Vault Using Ansible

“white wooden door surrounded by plants” by Daniel von Appen on Unsplash

Every application needs secret management which helps with authorization or authentication to the system using username password, database credentials, API tokens, TLS certificate, cloud provider access keys, and many other secrets. Developers used to share them through traditional methods like email, push them to the source code, keep them in configuration management like ansible, store them somewhere like S3 which creates secret sprawl. These methods have many flaws which prevent us from knowing who really have access to all these secrets, who accessed them in the past, the location of secret, how to rotate these secrets. We don’t want the whole world to know about these secrets. We want to keep them safe. If we want to expire secrets, we need to manually perform actions, create a new secret, distribute them and use them. We do not have any audit logs to track who requested a particular secret.

[HashiCorp Vault](https://www.vaultproject.io/) is a popular open source tool for secret management that allows users to store, manage and control access to tokens, username password, database credentials, TLS certificate, and many other secrets. Vault is used to manage secrets securely in a central location which prevents secret sprawl. Vault provides strict fine-grained control over who can access secrets. Value provide audit policies and audit logs on secrets which provide the strong guarantee and strong visibility into secret access. Vault handles leasing, key revocation, key rolling, and auditing. Vault supports pluggable mechanisms known as secrets engines for managing different secret types.

Applications can leak credentials through logs, diagnostic output like exception trace, shipping to an external monitoring tool etc. Vault provides dynamic secrets which help provide short-lived ephemeral credentials to applications instead of long live credentials. With dynamic secrets, even if an application writes it to an external system, it is valid for a specific period of time. We have different credentials for different services as they are dynamically fetched and unique, which help us identify exactly from where credential is used. This helps us in doing proper revocation, as we know what exact credential is compromised using audit logs. This prevents to reduce blast radius on credential compromise as no other service is effected when a specific credential is revoked.

Vault provides encryption as service. Vault shield encryption implementation from the end user and does key management instead of relying on the developer to do encryption using keys. This helps in ensuring cryptography is done in right way and key management with lifecycle management of keys.

Let’s list some of the benefits of using Vault:

1. Prevents secret sprawl

1. Provides dynamic secret

1. Encryption as a service

1. Provide audit logs

1. Revocation is easy

Let’s look in the setup of the vault using Ansible. We are going to create an Ansible Role for Vault setup so we can reuse it. We will begin by creating a new user account named “**vault**” which will help with a secure setup. We will use this account to isolate the ownership of vault. We don’t create any home directory or shell for this user so that user can’t log in to a server.

    - name: Creating vault user group
      group: 
        name: "{{ vault_group }}"
      become: true

    - name: Creating vault user
      user:
        name: "{{ vault_user }}"
        group: "{{ vault_group }}"
        system: yes
        shell: "/sbin/nologin"
        comment: "vault nologin User"
        createhome: "no"
        state: present

Next, we need to download vault archive from [here](https://www.vaultproject.io/downloads.html) on our remote vault instance. This will give a zip archive file. To unzip vault archive, we need to install unzip so we can unzip vault archive and takeout needed binary. Once this is done, we need to unzip vault archive, move our vault binary to “/usr/local/bin” and make vault user as the owner of this binary with needed permissions.

    - name: Install prerequisites
      package:
        name: "{{ item }}"
        update_cache: yes
      with_items: "{{ vault_install_prerequisites }}"
      become: yes

    - name: Download binary
      get_url:
        url: [https://releases.hashicorp.com/vault/{{vault_version}}/vault_{{vault_version}}_linux_amd64.zip](https://releases.hashicorp.com/vault/{{vault_version}}/vault_{{vault_version}}_linux_amd64.zip)
        dest: /tmp/vault_{{vault_version}}_linux_amd64.zip
        owner: "{{ vault_user }}"
        group: "{{ vault_group }}"
        mode: 0755
        checksum: "{{vault_checksum}}"
      register: vault_download

    - name: "Unzip vault archive"
      unarchive:
        src: "{{ vault_download.dest }}"
        dest: /usr/local/bin
        copy: no
        owner: "{{ vault_user }}"
        group: "{{ vault_group }}"
        mode: 0755

We need to set binary capabilities on Linux, to give the Vault executable the ability [to use the mlock syscall without running the process as root](https://www.vaultproject.io/docs/configuration/index.html).

    - name: "Set vault binary capabilities"
      capabilities:
        path: /usr/local/bin/vault
        capability: cap_ipc_lock+ep
        state: present

We need to setup systemd init file to manage the persistent vault daemon.

    [Unit]
    Description=Tool for managing secrets
    Documentation=[https://vaultproject.io/docs/](https://vaultproject.io/docs/)
    After=network.target
    ConditionFileNotEmpty=/etc/vault.hcl

    [Service]
    User=vault
    Group=vault
    ExecStart=/usr/local/bin/vault server -config=/etc/vault.hcl
    ExecReload=/usr/local/bin/kill --signal HUP $MAINPID
    CapabilityBoundingSet=CAP_SYSLOG CAP_IPC_LOCK
    Capabilities=CAP_IPC_LOCK+ep
    SecureBits=keep-caps
    NoNewPrivileges=yes
    KillSignal=SIGINT

    [Install]
    WantedBy=multi-user.target

We need to set above content into Systemd service file. Finally, start the vault server.

    - name: Copy systemd init file
      template:
        src: init.service.j2
        dest: /etc/systemd/system/vault.service
        owner: root
        group: root
      notify: systemd_reload

    - name: config file
      template:
        src: vault.hcl.j2
        dest: "{{ vault_config_path }}"
        owner: "{{ vault_user }}"
        group: "{{ vault_group }}"

    - name: vault service
      service:
        name: vault
        state: started
        enabled: yes

Vault starts in an uninitialised state. We need to set up initial parameters to unseal so it can be used. We need to set an environment variable so it is not needed to be provided on running vault commands. We are running vault in HTTP mode as being used by internal services. We can add TLS related changes by providing the certificate details in “vault.hcl” config file.

    export VAULT_ADDR=http://127.0.0.1:8200

First we are going to do complete process manually before doing it with Ansible for better understanding. We need to run init command “**vault operator init**” and get a response as the root token and unseal keys.

![](https://cdn-images-1.medium.com/max/3132/1*zyfNXb80uxvaM7mUdzgWzQ.png)

Once this is done, vault becomes initialised but remains seal. In seal state, it can’t store or provide any secret, more like locked unusable state. We need to unseal it with the required number of unseal keys. To provide keys we need to run command “**vault operator unseal**” and provide a unseal key.

![](https://cdn-images-1.medium.com/max/2048/1*R4f0DwxWDCtEAhp61t_2Hg.png)

Keep running this command, till we unseal vault successfully. In the default case, there are 5 unseal keys generated and we need to provide 3 of them.

![](https://cdn-images-1.medium.com/max/2276/1*4FtAeVYwas3MIoe2K2TRzg.png)

Once this unseal is done, we can check “**vault status**” to make sure it is unsealed.

![](https://cdn-images-1.medium.com/max/2224/1*UyCKKscgJ4NVNehCMbYsuQ.png)

Now lets do all this using ansible so we don’t need to do anything manually. Most important part is to make sure we have our unseal keys and root key secure as they are needed to unseal vault and do admin tasks. First, we are going to create unseal key and root key folder where we store our keys. We are going to create keys using “vault operation init” and register it in a variable. These keys then stored in our files so we can access them later.

    - name: Create unseal directories
      file:
        path: "{{ unseal_keys_dir_output }}"
        state: directory
      delegate_to: localhost

    - name: Create root key directories
      file:
        path: "{{ root_token_dir_output }}"
        state: directory
      delegate_to: localhost

    - name: Initialise Vault operator
      shell: vault operator init -key-shares=5 -key-threshold=3 -format json
      environment:
        VAULT_ADDR: "[http://127.0.0.1:8200](http://127.0.0.1:8200)"
      register: vault_init_results

    - name: Parse output of vault init
      set_fact:
        vault_init_parsed: "{{ vault_init_results.stdout | from_json }}"

    - name: Write unseal keys to files
      copy:
        dest: "{{ unseal_keys_dir_output }}/unseal_key_{{ item.0 }}"
        content: "{{ item.1 }}"
      with_indexed_items: "{{ vault_init_parsed.unseal_keys_hex }}"
      delegate_to: localhost

    - name: Write root token to file
      copy:
        content: "{{ vault_init_parsed.root_token }}"
        dest: "{{root_token_dir_output}}/rootkey"
      delegate_to: localhost

With this we have our vault init with unseal keys and root keys stored in local directory which can be used to unseal vault. We are going to divide this in two separate ansible roles **vault-init** and **vault-unseal** as we might need to unseal multiple times when vault goes down. To unseal, we need to read unseal keys and pass them during “**vault operator unseal**” command. Once this is done, we have our vault unsealed with ansible.

    - name: Reading unseal key contents
      command: cat {{item}}
      register: unseal_keys
      with_fileglob: "{{ unseal_keys_dir_output }}/*"
      delegate_to: localhost
      become: no

    - name: Unseal vault with unseal keys
      shell: |
        vault operator unseal {{ item.stdout }}
      environment:
        VAULT_ADDR: "[http://127.0.0.1:8200](http://127.0.0.1:8200)"
      with_items: "{{unseal_keys.results}}"

Now vault is ready for handling any secret management. We need to use root key for accessing vault which we got during the init process. Root token has root permissions to our vault deployment. We will set token as environment variable so we do not need to specify in each command.

    export VAULT_TOKEN=acd462e7-f8da-6554-6774-04c8b91aa520

We can test by saving a secret in the vault and then fetching the same secret to confirm that it is working as expected. We are writing the value as “secretData” for “secret/content”.

    vault write secret/content value=secretData

    Success! Data written to: secret/message

We read the value of “secret/content” from the vault and get “secretData” back which was stored previously. This confirms that it is working as expected.

    vault read secret/content

    Key                 Value

    ---                 -----

    refresh_interval    768h

    value               secretData

When vault service stops due to some reason we need to again unseal it with same unseal keys, so making sure we keep these unseal keys as safe as possible.

The complete code can be found in this git repository: [https://github.com/MiteshSharma/AnsibleVaultRole](https://github.com/MiteshSharma/AnsibleVaultRole)

Click here for “[Introduction To Ansible](https://medium.com/@mitesh_shamra/introduction-to-ansible-e5b56ee76b8c)”

Click here for “[Introduction To Ansible Roles](https://medium.com/@mitesh_shamra/ansible-roles-1d1954f9932a)”

***PS: If you liked the article, please support it with claps. Cheers***
