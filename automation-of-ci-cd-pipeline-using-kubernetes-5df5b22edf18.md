
# Automation of CI/CD Pipeline Using Kubernetes



**Kubernetes** (commonly stylized as **k8s**) is an open-source container orchestration system for automating application deployment, scaling, and management. It aims to provide a “platform for automating deployment, scaling, and operations of application containers across clusters of hosts”. It works with a range of container tools, including Docker. In other words, you can cluster together groups of hosts running Linux containers, and Kubernetes helps you easily and efficiently manage those clusters.

It helps you fully implement and rely on a container-based infrastructure in production environments. And because Kubernetes is all about automation of operational tasks, you can do many of the same things other application platforms or management systems let you do — but for your containers.

In order to meet changing business needs, your development team needs to be able to rapidly build new applications and services. Cloud-native development starts with microservices in containers, which enables faster development and makes it easier to transform and optimize existing applications.

In this article, I’m going to upgrade my previous article using Kubernetes.

Here is the article for your reference:
[**Fully Automation of CI/CD Pipeline**
*Automation is the ultimate need for DevOps practice and ‘Automate everything’ is the key principle of DevOps. In…*medium.com](https://medium.com/@shikharsrivastava_14544/fully-automation-of-ci-cd-pipeline-a671d163c940)

For this task, I have minikube already installed on my windows in a Virtual Machine(Oracle VM VBox) and RHEL8 on top of the VM which will be used as the workstation.

Step 1: Start minikube and get its IP.

* minikube start

* minikube ip

![](https://cdn-images-1.medium.com/max/2690/1*F_UXRJaXkmK93vk368n0GQ.png)

Step 2: Copy these three files from windows to rhel8 for configuration.

    C:\Users\KIIT\.minikube\ca.crt

    C:\Users\KIIT\.minikube\profiles\minikube\client.crt

    C:\Users\KIIT\.minikube\profiles\minikube\client.key

Step 3: Create a Jenkins image with kubectl installed on it. Configuration files will be given to it by mounting a directory.

Dockerfile:

![](https://cdn-images-1.medium.com/max/3840/1*k7-6hUbca9Ll6fHFQCJ-zA.png)

Now use the build command:

    docker build -t jenkin:kubectl .

Step 4: Now start the container, login and set up the Jenkins.

    docker run -it -v ~/kube-files:/root/.kube -P jenkin:kubectl

The contents of the config file :

    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority: ca.crt
        server: https://192.168.99.100:8443
    name: minikube
    contexts:
    - context:
        cluster: minikube
        user: minikube
      name: minikube
    current-context: minikube
    kind: Config
    preferences: {}
    users:
    - name: minikube
      user:
        client-certificate: client.crt
        client-key: client.key

Now our kubectl is completely configured on the rhel8 workstation.

Step 5: Job1 will remain the same as was in the previous article.

Step 6: Job2 execute shell commands.

![](https://cdn-images-1.medium.com/max/3840/1*wNxUj3oyCNLmTeR2uCHcaQ.png)

Step 7: Integrate Job3 with the following code to copy the files from GitHub to our pods.

    #!/usr/bin/python3
      
    import os

    cmd = os.popen("kubectl get pods | grep webserver")
    cmd_op = cmd.read()
    x = cmd_op.split('\n')
    del x[-1]
    for y in x:
        z = y.split()
        print(z)
        os.system("kubectl cp /root/.jenkins/workspace/job1/* {}:/usr/local/apache2/htdocs/".format(z[0]))

Step 8: Job4 will notify you using email if this build fails.

Now, we don’t require a monitoring job as this will be handled by k8s.

Now test it by connecting to minikube,

    C:\Users\KIIT\.minikube\profiles\minikube> kubectl get all
    NAME                             READY   STATUS    RESTARTS   AGE
    pod/webserver-85b3b5597d-78rwl   1/1     Running   0          4h
    pod/webserver-85b3b5597d-kvnzc   1/1     Running   0          4h
    pod/webserver-85b3b5597d-pwkzc   1/1     Running   0          4h

    NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        4h
    service/webserver    NodePort    10.96.68.216   <none>        80:32040/TCP   5h

    NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/webserver   3/3     3            3           4h

    NAME                                   DESIRED   CURRENT   READY   AGE
    replicaset.apps/webserver-85b3b5597d   3         3         3       4h

Now testing the webserver,

    C:\Users\KIIT>curl 192.168.99.100:32040

    Welcome to my website.
    How are you??

Thank you !!
