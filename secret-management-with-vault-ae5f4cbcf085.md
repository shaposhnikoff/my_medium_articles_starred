
# Secret management with Vault

Orchestration of sensitive information with Vault

Vault is a secret management tool; that centrally manages and distributes sensitive information. Vault is designed with the best practices in mind; passwords are safely stored and transported by design. **Vault** works by providing an API that is accessible by a service running as a client. The vault server provides through the API provides *sys*, *secret* and *auth* back-ends.

### Running vault

Running vault with an *in-mem* storage back end is great for testing. However for HA, it is advised to use consul; a **mysql** back end is not HA; if a vault instance is lost; even if a few existed, the leader vault cannot be easily discovered, unless a re-configuration occurs with the new vault IP. These is slower and more error prone. Vault with multiple **mysql** instances is not HA; it will connect to different databases and not share any cluster state with each other, which beats the purpose.

### Vault in HA mode with consul

Vault supports many back-ends; but they all do not provide high availability; consul and zookeeper support high availability. It is highly recommended to use consul to achieve HA. Consul is a storage back-end that gossips using the serf protocol. With this protocol; every cluster member sends a broadcast message to other members; notifying and updating state with the rest of the cluster. This allows for high availability and relatively strong consistency in the event of a network partition. To run vault in HA mode with consul — this is easy to configure and manage using **ansible**.

### DNS based Service discovery

When the vault service starts it registers itself with consul; identifying itself with a unique name. These services are exposed via a **RESTful** API or **DNS**; using a tool like **DNSmasq/BIND** this service names could be easily exposed on the host. It is advised to start vault with a list of consuls that would be discovered via DNS — i.e Sample **DNSmasq** configuration and *etc/resolv.conf *to add consul as a DNS server for the consul namespace.

**DNSmasq** resolves with consul which exposes service information through DNS.

Fetching secrets using *certificate authentication*; why is this necessary? This ensures that the client making the request is a legitimate client and is among the hosts allowed to make the request. Firewall rules could be used; in this case IP spoofing will be a risk that needs to be dealt with. Vault support other auth back-ends such as *token*, *appid* or *user&pass. *Client certificates ate the most robust IMO; since they can be dynamically revoked.

### Managing vault

Vault is a secret store and to prevent leaks it should be *sealed *through the API, when an IDS has detected an intrusion is occurring vault should be sealed (the IDS could trigger a seal). **Vault **can be hard to unseal; it needs *unsealing *whenever a service or machine restart is initiated. This could also be done through the API; the unseal keys could be between three to as many keys as the organisation needs. Three should be sufficient; since these keys will be kept by a group of administrators.

### Deploying vault

Watch out for HA, use a HA available data store. Service discovery through DNS, and cache poisoning. You don’t want to run vault in non-*HA* mode and have your services not start white trying to pull secrets / *updating config*. This could cause a massive outage; especially if secrets are periodically pulled.

### Auditing

Audit back-ends such as files can be very effective; when properly consumed. They are simple and can be consumed and analysed.

### API

The Vault API is REST*full* and [easy](https://www.vaultproject.io/docs/http/) to work with. This API allows a service to automate and be in control of sensitive issues such as Vault *unsealing* / *sealing, and leader stand-down.*

### Sample client request

In this sample request, we present a *certificate* and *keyfile*; the response is a one time *token. *This OTT is used to make the request to the vault secret back-end; the certificate allows us to generate a token bound to the policy the application is allowed to access.

    *# Using Python for this example, should be similar in other languages*
    **import** requests
    **import** json
    
    **from** requests.packages.urllib3.exceptions **import** InsecureRequestWarning
    requests**.**packages**.**urllib3**.**disable_warnings(InsecureRequestWarning)
    
    cert_file_path **=** "/etc/cert/proj.crt"
    key_file_path **=** "/etc/cert/proj.key"
    
    cert **=** (cert_file_path, key_file_path)
    
    r **=** requests**.**post('https://active.vault.consul:8200/v1/auth/cert/login', cert**=**cert, verify**=**False)
    json_response **=** r**.**json()
    
    **if** r**.**status_code **==** 200:
        token **=** json_response**.**get('auth',{})
        client_token **=** token**.**get('client_token', None)
    
        headers **=** {"X-Vault-Token": "{}"**.**format(client_token)}
        r **=** requests**.**get('https://active.vault.consul:8200/v1/secret/proj', verify**=**False, headers**=**headers)
        json_response **=** r**.**json()
        **print** json**.**dumps(json_response)

### References

Consul [1] [https://www.consul.io/docs/index.html](https://www.consul.io/docs/index.html)

Vault [2] [https://www.vaultproject.io/docs/](https://www.vaultproject.io/docs/)
