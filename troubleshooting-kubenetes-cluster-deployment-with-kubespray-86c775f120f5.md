
# Troubleshooting Kubenetes Cluster Deployment with Kubespray

On successful run of Kubespray necessary Kubernetes Control Plane components and other system related components will be installed in master and worker nodes in order to properly functioning of kubernetes cluster.

But during deployment you will get multiple issues which will make the installation setup very daunting. I’ll list down here challenges i faced during deployment of cluster and what i did to troubleshoot issues.

1. If process struck at task **Kubernetes Apps | Wait for kube-apiserver **then it certainly means kubelet service is not running properly in master node and you need to figure out the reason why kubelet service is not running. kubelet service uses** **/**etc/systemd/system/kubelet.service.d/10-kubeadm.conf** configuration to bootup kubelet service. check whether the configuration is correct or not and make sure kubelet service should be up and running in order to pass this task.

1. If nodes are running behind a proxy then docker container will not pull images properly then you need to create **http-proxy.conf** manually in path **/etc/systemd/system/docker.service.d/** with following values as per your environment to work docker properly.
root@master:~/kubespray# cat /etc/systemd/system/docker.service.d/https-proxy.conf
[Service]
Environment=”HTTP_PROXY=[http://<proxy_ip>:<proxy:port>/](http://165.225.104.34:9480/)"
Environment=”HTTPS_PROXY=[http://<proxy_ip>:<proxy:port>](http://165.225.104.34:9480/)[/](http://165.225.104.34:9480/)"
Environment=”NO_PROXY=<master_node_ip>,<worker_node_ip>,10.96.0.0/12,localhost,127.0.0.1,192.168.0.0/16"
then restart again then it will work.

1. If process struck at task file_download | Download item and you are running behind a proxy then you need to update this task as following:
- name: file_download | Download item
 get_url:
 url: “{{download.url}}”
 dest: “{{download.dest}}”
 sha256sum: “{{download.sha256 | default(omit)}}”
 owner: “{{ download.owner|default(omit) }}”
 mode: “{{ download.mode|default(omit) }}”
 validate_certs: “{{ download_validate_certs }}”
 url_username: “{{ download.username|default(omit) }}”
 url_password: “{{ download.password|default(omit) }}”
 force_basic_auth: “{{ download.force_basic_auth|default(omit) }}”
 use_proxy: yes
 environment:
 http_proxy: [http://<proxy_ip>:](http://165.225.104.34:9480)<proxy_port>
 https_proxy: [http://<proxy_ip>:](http://165.225.104.34:9480)<proxy_port>
 register: get_url_result
 until: “‘OK’ in get_url_result.msg or ‘file already exists’ in get_url_result.msg”
 retries: 4
 delay: “{{ retry_stagger | default(5) }}”
 when:
 — download.enabled
 — download.file
 — group_names | intersect(download.groups) | length
then run again and it will install items from proxy.

1. If process struck at task **kubeadm | Initialize first master **then it’s ssl certificates issue. Kubeadm expects certificates to be present inside **/etc/kubernetes/pki** directory but kubespray places certificates inside **/etc/kubernetes/ssl** directory. in this case you need to create pki directory and copy CA certificates there.
**cp /etc/kubernetes/ssl/ca.crt /etc/kubernetes/pki/ca.crt
cp /etc/kubernetes/ssl/ca.key /etc/kubernetes/pki/ca.key
**then run playbook again and it will work.

1. If process struck at task **Join to cluster with async timeout errors **then you need to copy **/etc/kubernetes/kubelet.conf **file from master node to worker node and restart kubelet service on worker node then only it will work.

1. If above task successful then still worker node is not able to join the cluster then issue is with Kube api server admission control. to fix this issue open **/etc/kubernetes/manifests/kube-apiserver.yaml and remove flag — enable-admission-control=NodeRestriction and — authorization-mode=RBAC. **then using kubectl get nodes you will be able to see master and worker nodes.

1. If process struck with task **TASK [win_nodes/kubernetes_patch : Check current nodeselector for kube-proxy daemonset]** then it means Kubespray didn’t install kube-proxy as daemonset and you need to install it manually and set the cluster role binding for kube-proxy manually. You can follow this issue to fix this issue:
 [https://github.com/kubernetes-sigs/kubespray/issues/4600](https://github.com/kubernetes-sigs/kubespray/issues/4600)
this fix will properly configure kube-proxy on master and worker nodes.
If kube-proxy logs are throwing error something like **clusterCidr missing **then get cluster cidr from **kubectl cluster-info dump| grep cluster-cidr** and add flag in **kube-proxy-daemonset.yaml** as command flag **— — cluster-cidr=<cluster_cidr> **and apply daemonset.

1. If process struck with task **Flannel | Wait for flannel subnet.env** file presence then you need to change flannel version here from **“v0.11.0”** to **“v0.11.0-amd64”** and restart playbook again. it will then install subnet.env in **/run/flannel** directory.

1. If all tasks are success but coredns pods continuous failing then make following changes in this file.
root@master:~/kubespray# cat roles/kubernetes-apps/ansible/templates/coredns-config.yml.j2
 — -
apiVersion: v1
kind: ConfigMap
metadata:
 name: coredns
 namespace: kube-system
 labels:
 addonmanager.kubernetes.io/mode: EnsureExists
data:
 Corefile: |
 .:53 {
 errors
 health
 kubernetes {{ dns_domain }} in-addr.arpa ip6.arpa {
 pods insecure
{% if resolvconf_mode == ‘host_resolvconf’ and upstream_dns_servers is defined and upstream_dns_servers|length > 0 %}
 upstream {{ upstream_dns_servers|join(‘ ‘) }}
{% else %}
 upstream 8.8.8.8 8.8.4.4
{% endif %}
 fallthrough in-addr.arpa ip6.arpa
 }
 prometheus :9153
{% if resolvconf_mode == ‘host_resolvconf’ and upstream_dns_servers is defined and upstream_dns_servers|length > 0 %}
 forward . {{ upstream_dns_servers|join(‘ ‘) }}
{% else %}
 forward . 8.8.8.8 8.8.4.4
{% endif %}
 cache 30
 loop
 reload
 loadbalance
 }
this will resolve coredns DNS Loop issue as it points to upstreams 8.8.8.8 and 8.8.4.4 which are google DNS name servers.

1. If process struck during helm installation at task Helm | Install/upgrade helm and you are running behind a proxy then you need to change file as per your proxy settings:
root@master:~/kubespray# cat roles/kubernetes-apps/helm/templates/helm-container.j2
#!/bin/bash
{{ docker_bin_dir }}/docker run — rm \
 — net=host \
 — name=helm \
 -v /root/.kube:/root/.kube:ro \
 -v /etc/ssl:/etc/ssl:ro \
 -v {{ helm_home_dir }}:{{ helm_home_dir }}:rw \
 {% for dir in ssl_ca_dirs -%}
 -v {{ dir }}:{{ dir }}:ro \
 {% endfor -%}
 {% if http_proxy is defined or https_proxy is defined -%}
 -e http_proxy=”[http://<proxy_ip>:](http://165.225.104.34:9480)<proxy_port>" \
 -e https_proxy=”[http://<proxy_ip>:<](http://165.225.104.34:9480)proxy_port>" \
 -e no_proxy=”{{proxy_env.no_proxy}}” \
 {% endif -%}
 {{ helm_image_repo }}:{{ helm_image_tag}} \
 “$@”
then again run playbook and you can see tiller as pod.

1. If even after successful cluster deployment pod to pod and pod to service communication is not happening then check **coredns** pod logs. if there is any unusual error coredns is throwing fix them otherwise get coredns **cluster ip** using **kubectl get svc | grep coredns** and go to **/var/lib/kubelet/config.yaml** and check value of clusterDNS it must be same as coredns cluster ip and if it is not replace clusterDNS value with coredns cluster ip and restart kubelet service and redeploy pods again then pod to pod and pod to service communication will work possibly. Reason behind this is kubelet creates /etc/resolv.conf inside each pod which pod will use to communicate with cluster.

1. If docker needs to install containers from insecure or private registry then it needs to be added in all nodes in /etc/docker/daemon.json like as below
root@master:~# cat /etc/docker/daemon.json
{
 “exec-opts”: [“native.cgroupdriver=cgroupfs”],
 “insecure-registries” : [“<registry_ip:port>”]
}
after adding this restart docker daemon
systemctl daemon-reload && systemctl restart docker

1. If process struck at task** ensure docker-ce repository public key is installed** it means docker is not getting installed in the nodes** **then manually docker gpg keys need to be added in apt-cache policy by following steps:
curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo apt-key add -
sudo add-apt-repository “deb [arch=amd64] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) $(lsb_release -cs) stable”
sudo apt-get update
apt-cache policy docker-ce
after this run the playbook again.

If you encounter any issues which are not mentioned above then please comment here. I’ll be happy to help.

Thanks!!!
