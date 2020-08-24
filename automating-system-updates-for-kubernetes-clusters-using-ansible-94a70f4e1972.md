Unknown markup type 10 { type: [33m10[39m, start: [33m109[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m444[39m, end: [33m453[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m538[39m, end: [33m554[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m14[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m167[39m, end: [33m183[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m9[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m87[39m, end: [33m98[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m143[39m, end: [33m159[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m229[39m, end: [33m240[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m374[39m, end: [33m390[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m406[39m, end: [33m416[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m9[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m80[39m, end: [33m90[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m160[39m, end: [33m172[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m237[39m, end: [33m247[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m257[39m, end: [33m277[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m403[39m, end: [33m417[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m53[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m84[39m, end: [33m98[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m122[39m, end: [33m133[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m214[39m, end: [33m230[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m405[39m, end: [33m418[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m143[39m, end: [33m159[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m378[39m, end: [33m402[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m619[39m, end: [33m635[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m652[39m, end: [33m663[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m725[39m, end: [33m732[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m119[39m, end: [33m137[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m151[39m, end: [33m167[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m6[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m60[39m, end: [33m75[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m134[39m, end: [33m143[39m }

# Automating System Updates for Kubernetes Clusters using Ansible



**Kubespray** is the go to tool for deploying self managed Kubernetes clusters. Built on **Ansible**, Kubespray makes it simple to deploy, update, and expand Kubernetes clusters. I highly recommend Kubespray if you are deploying your own bare-metal Kubernetes cluster or if you want to save money by not using cloud services like **GKE** (Google Cloud), **EKS** (AWS) or **AKS** (Azure).

Kubespray provides plenty of tools for managing Kubernetes, but it doesn’t provide any tools for managing the underlying infrasture. In this post I am going to show how use Ansible to automate zero down time system updates for Kubernetes clusters. The process for completing system updates on Kubernetes clusters is to:

1. **Cordon** the node so no new pods are scheduled on the node

1. **Drain** the node so all of the existing workloads are moved to other nodes

1. U**pdate and reboot** the node

1. Finally **uncordon** the node so new pods can be scheduled on the node

This method can also be used to automate other maintenance tasks on any type of cluster with out causing any down time for your servers.

I will start with the Ansible Playbook. There are two important parts in defining the Playbook. First is the serialsetting. Normally Ansible executes the tasks on all of the nodes in the inventory in parallel, however updating and rebooting all of the nodes at once would take all of the nodes offline at once, interrupting services and causing the Kubernetes cluster to enter a potentally irrecoverable bad state. To avoid this issue, setting serial: 1 will cause the Ansible Playbook to run on one node at a time. The second setting is any_errors_fatal**. **Depending on the size of your cluster or how critical it is, you may (or may not) want to halt the operation of the Ansible Playbook if there is an error.

Here is my playbook.yml file:

    ---
    - hosts:
        - kube-master
        - kube-node
      become: true
      become_method: sudo
      **serial: 1**
      **any_errors_fatal: "{{ any_errors_fatal | default(true) }}"**
    
      roles:
        - k8s-rolling-update

After setting serial: 1 the Ansible Playbook will run the k8s-rolling-update role in a loop, one node at a time. Now we will look at the role.

First of all, we want to check each node and make sure Kubernets reports the node is in the **Ready** state and is **Uncordoned**. We will use Ansible’s [command module](https://docs.ansible.com/ansible/latest/modules/command_module.html) to run kubectl get node and parse the JSON output. We will run kubectl via the command module instead of the [k8s_info module](https://docs.ansible.com/ansible/latest/modules/k8s_info_module.html) because the k8s_info module requires the OpenShift Python client installed on each node. The OpenShift Python client isn’t used by Kubespray and isn’t commonly available.

Below is tasks/main.yml. This task runs kubectl get node only on the first node in the kube-master inventory group and saves the output to the kubectl_get_node variable. We want this command to only run on one node so we use the delegate_to option. Then it parses the JSON output and if the node is **Ready** and **Uncordoned** the task will run 3 other tasks. The JSON output from kubectl get node is complex, so [json_query](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#json-query-filter)filter is used to parse the JSON. The json_query filter uses [jmespath](https://jmespath.org) and can accept jmespath queries.

drain.yml will complete the cordon and drain (steps 1 & 2 from the above list), ubuntu.yml will complete the update and reboot (step 3 from the above list) and uncordon.yml will uncordon the node (step 4 from the above list). I refer to ubuntu.yml with the ansible_distribution variable, this way future updates of this role could include update tasks for different Linux distributions without updating tasks/main.yml.

    ---
    - name: Get the node's details
      command: >-
        {{ bin_dir }}/kubectl get node
        {{ kube_override_hostname|default(inventory_hostname) }}
        -o json
      register: kubectl_get_node
      delegate_to: "{{ groups['kube-master'][0] }}"
      failed_when: false
      changed_when: false
    
    - name: Update Node
      when:
        *# When status.conditions[x].type == Ready then check stats.conditions[x].status for True|False
        *- kubectl_get_node['stdout'] | from_json | json_query("status.conditions[?type == 'Ready'].status")
        *# If spec.unschedulable is defined then the node is cordoned
        *- not (kubectl_get_node['stdout'] | from_json).spec.unschedulable is defined
      block:
        - name: Cordon & drain node
          include_tasks: **drain.yml**
    
        - name: Upgrade the Operating System
          include_tasks: **"{{ ansible_distribution }}.yml"**
    
        - name: Uncordon node
          include_tasks: **uncordon.yml**

Now for the Drain and Cordon tasks in tasks/drain.yml. The first task is to run the kubectl cordon command from the first kube-master node to cordon the working node from the rest of the cluster. Next, we will run kubectl get node again to verify that the node has been cordoned. This task will retry 10 times waiting 10 second in between until the node has been cordoned. Finally, the last task will run kubectl drain on the working node to evict any running pods so it is safe to upgrade Docker on the node or reboot the node, which we will do in the next step.

    ---
    - name: Cordon node
      command: >-
        {{ bin_dir }}/**kubectl cordon**
        {{ kube_override_hostname|default(inventory_hostname) }}
      delegate_to: "{{ groups['kube-master'][0] }}"
    
    - name: Wait for node to cordon
      command: >-
        {{ bin_dir }}/**kubectl get node**
        {{ kube_override_hostname|default(inventory_hostname) }}
        -o json
      register: wait_for_cordon
      **retries: 10**
      **delay: 10**
      delegate_to: "{{ groups['kube-master'][0] }}"
      changed_when: false
      until: (wait_for_cordon['stdout'] | from_json).spec.unschedulable
    
    - name: Drain node
      command: >-
        {{ bin_dir }}/**kubectl drain**
        --force
        --ignore-daemonsets
        --grace-period {{ drain_grace_period }}
        --timeout {{ drain_timeout }}
        --delete-local-data {{ kube_override_hostname|default(inventory_hostname) }}
      delegate_to: "{{ groups['kube-master'][0] }}"

After all of the pods have been drained off of the node, you can run any tasks on the node without interrupting any services. In this example, tasks/ubuntu.yml will update all of the packages on the node and reboot the server if necessary. First, the [apt module](https://docs.ansible.com/ansible/latest/modules/apt_module.html) updates all packages on the node. Then the task checks to see if a reboot is needed after the update by checking if /var/run/reboot-required exists. If /var/run/reboot-required exists, then the node is rebooted using the [reboot module](https://docs.ansible.com/ansible/latest/modules/reboot_module.html). The reboot module will restart the node, wait for the code to come back online before proceeding to the next task. **Note:** tasks/ubuntu.yml doesn’t use the delegate_to for any of the tasks, we want to delegate the running of the kubectl command to one node, but we want run the updates and the reboots on the nodes when it is their turn.

    ---
    - name: Update all packages
      apt:
        upgrade: dist
        update_cache: true
        force_apt_get: true
    
    - name: Check if reboot is required
      stat:
        path: /var/run/reboot-required
      register: reboot_required
    
    - name: Reboot the server
      reboot:
        post_reboot_delay: 30
      when: reboot_required.stat.exists

Finally, once the node has been updated and rebooted, we want to uncordon the node so new pods can be scheduled on it. tasks/uncordon.yml will run the kubectl uncordon command on updated the node, then verify that the node is indeed scheduleable.

    ---
    - name: Uncordon node
      command: >-
        {{ bin_dir }}/kubectl uncordon
        {{ kube_override_hostname|default(inventory_hostname) }}
      delegate_to: "{{ groups['kube-master'][0] }}"
    
    - name: Wait for node to uncordon
      command: >-
        {{ bin_dir }}/kubectl get node
        {{ kube_override_hostname|default(inventory_hostname) }}
        -o json
      register: wait_for_uncordon
      retries: 10
      delay: 10
      delegate_to: "{{ groups['kube-master'][0] }}"
      changed_when: false
      until: not (kubectl_get_node['stdout'] | from_json).spec.unschedulable is defined

After tasks/uncordon.yml is done running on the first node, tasks/drain.yml will begin on the second node and so on, again due to the serial: 1 setting in the Playbook. That is it! You can use and expand this example to work do any task on any type of cluster, the same method should work for **Hadoop** and **Spark** clusters or even **Mysql** and **Postgres** database replica.

You can view the working role at [https://github.com/kevincoakley/ansible-role-k8s-rolling-update](https://github.com/kevincoakley/ansible-role-k8s-rolling-update) .
