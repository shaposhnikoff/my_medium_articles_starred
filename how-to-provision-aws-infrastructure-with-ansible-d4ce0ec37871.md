Unknown markup type 10 { type: [33m10[39m, start: [33m161[39m, end: [33m170[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m3[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m195[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m156[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m82[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m85[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m157[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m175[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m180[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m175[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m233[39m }

# How to Provision AWS Infrastructure with Ansible?

Ansible

### Preparing a local environment

As you most likely already grasp, Ansible is an orchestration tool that enables you to write plain-text playbook files that declare the code profile and ideal state you want to be applied to a target server. Those servers — called hosts — are provisioned for concerning any digital work you’ll imagine, exploitation concerning any combination of application code, and running on concerning any platform.

In the smart previous days, once a playbook was run against a physical server, Ansible would use Associate in Nursing existing SSH association to firmly login to the remote host and set about building your application. however, that will not work for AWS workloads. You see, as a result of the EC2 instances and alternative infrastructure you would like to launch do not however exist, there are no “existing” SSH connections. Instead, Ansible can use Boto three — the code development kit (or SDK) employed by AWS that permits Python code to speak with the AWS API.

### Using the AWS **interface** **to attach** Ansible

You don’t ought to savvy all that works, however, it’s to be there, therefore, it will work. For that reason, you are going to put in the AWS interface (CLI). we cannot be using the interface itself for all the world vital, however, putting in it’ll offer us all the dependencies we’ll like. you’ll be able to ascertain a way to create this work on the most recent version of no matter the OS you are using from the AWS documentation page.

Working with the Python package manager, PIP, could be fashionable thanks to getting all this done. Here’s however you’d install PIP itself and so the AWS interface on Ubuntu machine:

## 1. How to launch EC2 Instance with Ansible

**Pre-requisite**

* Make sure you have Ansible and boto installed in your Mac/Laptop

* Boto will hold the Access Key and Secret Key details, which is required by Ansible

**Let create a boto file**

    vim ~/.boto

    [Credentials]
    aws_access_key_id = YOUR ACCESS KEY
    aws_secret_access_key = YOUR SECRET KEY

**Lets create inventory file**

This will be needed for Ansible to read the inventory file while launching the ec2-instances, also a new server which will be launched under a group name called webserver.

    vim hosts

    [local]
    localhost

    [webserver]

**Make sure you have your private key file in the same directory to make an SSH connection**

I am using the key name as DevOps-ansible, this key name might be different in your environment.

**Create a file to launch ec2 instances**

Sample code where you can launch t2.micro instance of Ubuntu.

**Let run the playbook**

    ansible-playbook -i hosts ec2_launch.yml

You will see the output something like this, and you can verify the public ip would be added in your host file

    PLAY [Provision an EC2 Instance] ****************************************************************************************************************************

    TASK [Create a security group] ******************************************************************************************************************************
    ok: [localhost -> localhost]

    TASK [Launch the new EC2 Instance] **************************************************************************************************************************
    changed: [localhost -> localhost]

    TASK [Add the newly created EC2 instance(s) to the local host group (located inside the directory)] *********************************************************
    changed: [localhost -> localhost] => (item={u'ramdisk': None, u'kernel': None, u'root_device_type': u'ebs', u'private_dns_name': u'ip-172-31-87-188.ec2.internal', u'block_device_mapping': {u'/dev/sda1': {u'status': u'attached', u'delete_on_termination': False, u'volume_id': u'vol-053f496491bfe5f37'}}, u'key_name': u'devops-ansible', u'public_ip': u'3.85.47.231', u'image_id': u'ami-0f9cf087c1f27d9b1', u'tenancy': u'default', u'private_ip': u'172.31.87.188', u'groups': {u'sg-0e40ed94232568846': u'webserver'}, u'public_dns_name': u'ec2-3-85-47-231.compute-1.amazonaws.com', u'state_code': 16, u'id': u'i-06e6b76acb57407a3', u'tags': {}, u'placement': u'us-east-1d', u'ami_launch_index': u'0', u'dns_name': u'ec2-3-85-47-231.compute-1.amazonaws.com', u'region': u'us-east-1', u'ebs_optimized': False, u'launch_time': u'2019-02-09T12:22:59.000Z', u'instance_type': u't2.micro', u'state': u'running', u'architecture': u'x86_64', u'hypervisor': u'xen', u'virtualization_type': u'hvm', u'root_device_name': u'/dev/sda1'})

    TASK [Wait for SSH to come up] ******************************************************************************************************************************
    ok: [localhost -> localhost] => (item={u'ramdisk': None, u'kernel': None, u'root_device_type': u'ebs', u'private_dns_name': u'ip-172-31-87-188.ec2.internal', u'block_device_mapping': {u'/dev/sda1': {u'status': u'attached', u'delete_on_termination': False, u'volume_id': u'vol-053f496491bfe5f37'}}, u'key_name': u'devops-ansible', u'public_ip': u'3.85.47.231', u'image_id': u'ami-0f9cf087c1f27d9b1', u'tenancy': u'default', u'private_ip': u'172.31.87.188', u'groups': {u'sg-0e40ed94232568846': u'webserver'}, u'public_dns_name': u'ec2-3-85-47-231.compute-1.amazonaws.com', u'state_code': 16, u'id': u'i-06e6b76acb57407a3', u'tags': {}, u'placement': u'us-east-1d', u'ami_launch_index': u'0', u'dns_name': u'ec2-3-85-47-231.compute-1.amazonaws.com', u'region': u'us-east-1', u'ebs_optimized': False, u'launch_time': u'2019-02-09T12:22:59.000Z', u'instance_type': u't2.micro', u'state': u'running', u'architecture': u'x86_64', u'hypervisor': u'xen', u'virtualization_type': u'hvm', u'root_device_name': u'/dev/sda1'})

    TASK [Add tag to Instance(s)] *******************************************************************************************************************************
    changed: [localhost -> localhost] => (item={u'ramdisk': None, u'kernel': None, u'root_device_type': u'ebs', u'private_dns_name': u'ip-172-31-87-188.ec2.internal', u'block_device_mapping': {u'/dev/sda1': {u'status': u'attached', u'delete_on_termination': False, u'volume_id': u'vol-053f496491bfe5f37'}}, u'key_name': u'devops-ansible', u'public_ip': u'3.85.47.231', u'image_id': u'ami-0f9cf087c1f27d9b1', u'tenancy': u'default', u'private_ip': u'172.31.87.188', u'groups': {u'sg-0e40ed94232568846': u'webserver'}, u'public_dns_name': u'ec2-3-85-47-231.compute-1.amazonaws.com', u'state_code': 16, u'id': u'i-06e6b76acb57407a3', u'tags': {}, u'placement': u'us-east-1d', u'ami_launch_index': u'0', u'dns_name': u'ec2-3-85-47-231.compute-1.amazonaws.com', u'region': u'us-east-1', u'ebs_optimized': False, u'launch_time': u'2019-02-09T12:22:59.000Z', u'instance_type': u't2.micro', u'state': u'running', u'architecture': u'x86_64', u'hypervisor': u'xen', u'virtualization_type': u'hvm', u'root_device_name': u'/dev/sda1'})

    PLAY RECAP **************************************************************************************************************************************************
    localhost                  : ok=5    changed=3    unreachable=0    failed=0

**Hola your ec2-instance is ready NOW !!!**

## Destroy you Setup !! < Do not do this in Production >

This will help you to tear down your test setup which you created for testing.

This will match the TAG set as Name: webserver and terminate those instances.

    ---

    - hosts: local
      connection: local
      vars:
        region: us-east-1
      tasks:
        - name: Gather EC2 facts
          ec2_instance_facts:
            region: "{{ region }}"
            filters:
              "tag:Name": "webserver"
          register: ec2
        - debug: var=ec2

        - name: Terminate EC2 Instance(s)
          ec2:
            instance_ids: '{{ item.instance_id }}'
            state: absent
            region: "{{ region }}"
          with_items: "{{ ec2.instances }}"

## 2. Now, Lets Install Software to the AWS infrastructure

Pre-requisites

* Assuming you have all nodes connected and you're able to ping pong the nodes

**What we will do**

* Create an index.html and copy to destination node

* Install apache latest apache package

* Once file is copied restart the apache service to serve the new index.html file

**Playbook, create a directory as playbooks, and create the below files.**

* vim install_apache.yml

    ---

    - name: Install apache packages
      hosts: web
      become: true
      tasks:
        - name: Install apache packages
          apt:
            name:
            - apache2
            update_cache: yes
            state: latest

        - name: copy default index.html to /var/www/html
          copy:
            src: index.html
            dest: /var/www/html/
          notify:
            - restart apache

      handlers:
        - name: restart apache
          service: name=apache2 state=restarted

* vim index.html

    Hola Ansible is working !!!

* Make sure the index.html should be in same directory of playbook

**Let us run the playbook**

    ansible-playbook playbook/install_apache.yml

**Output of playbook**

    ubuntu@ip-172-31-89-135:~/ansible-demo$ ansible-playbook playbooks/install_apache.yml

    PLAY [Install apache packages] ******************************************************************************************************************************

    TASK [Gathering Facts] **************************************************************************************************************************************
    ok: [172.31.80.9]

    TASK [Install apache packages] ******************************************************************************************************************************
    changed: [172.31.80.9]

    TASK [copy default index.html to /var/www/html] *************************************************************************************************************
    ok: [172.31.80.9]

    PLAY RECAP **************************************************************************************************************************************************
    172.31.80.9                : ok=3    changed=1    unreachable=0    failed=0

* Once you have successfully run the playbook, browse the web interface and you will see the content of index.html which you created earlier.

Visit here for the [code](https://github.com/sanjaynaikwadi/ansible)

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Subscribe to [FAUN topics](https://www.faun.dev/join?utm_source=medium.com/faun&utm_medium=medium&utm_campaign=faunmediumprebanner) and get your weekly curated email of the must-read tech stories, news, and tutorials **🗞️

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬

![](https://cdn-images-1.medium.com/max/3000/1*_cT0_laE4iPcqW1qrbstAg.gif)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
