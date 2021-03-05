
# Scalable Microservice Demo K8s Istio Kafka

Demonstration of a highly scalable microservice application with asynchronous communication using Kafka

![[https://unsplash.com/photos/-rckGTtnN7w](https://unsplash.com/photos/-rckGTtnN7w)](https://cdn-images-1.medium.com/max/12000/1*nod1-jbkmrfcJB987Zm9CA.jpeg)*[https://unsplash.com/photos/-rckGTtnN7w](https://unsplash.com/photos/-rckGTtnN7w)*

### Series content

This series creates the same Scalable Microservice Application using different technologies:

1. (this article)

1. [Scalable Serverless Microservice Demo using AWS Lambda Kinesis](https://medium.com/@wuestkamp/scalable-serverless-microservice-demo-aws-lambda-kinesis-terraform-cbe6036bf5ac?source=friends_link&sk=074614683a6641cab9b6067929bdc660)

1. Scalable Serverless Microservice Demo using Knative and Kafka (planned)

### What’s this about?

This describes a Highly Scalable Microservice Demo application using Kubernetes, Istio and Kafka. Through synchronous REST API calls it’s possible to create users. Inside, all communication is done asynchronously through Kafka.

![Image 1: Architecture overview](https://cdn-images-1.medium.com/max/2896/1*8yh_ZFZs5J17RCJ7Eevgqw.png)*Image 1: Architecture overview*

The Kafka consumer/producer UserApprovalService is automatically scaled (HPA) based on how many unhandled messages are in the Kafka topic. There is also a Node/Cluster scaler in place.

**We will scale up to 23000 Kafka events per second, 11 Kubernetes nodes and 280 pods.**

![Image 2: Results overview](https://cdn-images-1.medium.com/max/3568/1*j9_jyDzW2xymZzbC_xi_AQ.png)*Image 2: Results overview*

The application is completely scripted with the help of Terraform and can be run using one single command.

### Technology stack:

* Terraform

* (Azure) K8s, MongoDB, Container Registry

* (ConfluentCloud) Kafka

* Istio

* Grafana, Prometheus, Kafka Exporter, Kiali

* Python, Go

### Repository

[https://github.com/wuestkamp/scalable-microservice-demo](https://github.com/wuestkamp/scalable-microservice-demo)

Check the readme for instructions on how to run it yourself.

## Architecture

![Image 3: Architecture](https://cdn-images-1.medium.com/max/2796/1*T2dd4tVz1zQci2cj8NMdIw.png)*Image 3: Architecture*

We have three microservices:

* **OperationService** (Python): receives synchronous REST requests and converts these into asynchronous events. Keeps track of requests as “operations” and stores these in its own database.

* **UserService** (Python): handles user creation and stores these in its own database.

* **UserApprovalServer** (Go): can approve/deny a user, stateless.

The Kafka cluster is managed by ConfluentCloud, the Mongo databases and the K8s cluster are managed by Azure.

### Database per Service Pattern

We don’t use one large database which multiple services share, every service has its own database if it's stateful. We still only have one MongoDB database server, but on that one server can exist multiple databases. Microservices can share the same database server if they use the same type/version. Read more [here](https://microservices.io/patterns/data/database-per-service.html).

### Asynchronous communication

The three microservices communicate asynchronously with each other, there is no direct synchronous connection. One advantage of async is loose coupling. If the UserApprovalService is down for some time, the request doesn’t fail but just takes longer till the user gets approved. Hence when using async communication there is no need for implementing retries or circuit breakers.

### Message Workflow

![Image 4: Message workflow](https://cdn-images-1.medium.com/max/2932/1*E_kDrSMB-Vlzp7Y9OFh72w.png)*Image 4: Message workflow*

Image 4 shows the messages produced and consumed. The **UserService** consumes user-create messages, creates “pending-approval” users stored in MongoDB and produces the message user-approve.

Once it receives the user-approve-response message from **UserApprovalService**, it updates the user to “approved” or “not-approved” and produces the user-create-response message, which will be received by the **OperationService** which will update the operation status to “completed”.

### SAGA Pattern

When you work with one large (MySQL) relational database you can simply wrap your actions in database transactions. The SAGA pattern can be used to implement ACID like transactions for actions across multiple microservices.

In Image 4 the UserService could be considered as the orchestrator of the SAGA user-create. Because it coordinates the user creation through producing and consuming various messages. In this example, only one more service (UserApprovalService) is involved, but this could get more complex with many more services.

A SAGA can be compared to and implemented as a State Machine. To read more about the SAGA pattern and the difference between Orchestration and Choreography here: [https://microservices.io/patterns/data/saga.html](https://microservices.io/patterns/data/saga.html)

### Synchronous <-> Asynchronous conversion

![Image 5: Sync to Async conversion](https://cdn-images-1.medium.com/max/2656/1*yDFDndNhzL1mXjg9tXqqzw.png)*Image 5: Sync to Async conversion*

(**1**) Image 5 shows that at first a synchronous REST call is made to the OperationService to create a new operation, in this case “user-create”. The OperationService emits an asynchronous message and then immediately returns the new operation in a pending state.

(**2**) The returned operation contains a UUID which can then be used to periodically fetch the current state of that operation. The operation will be updated based on further asynchronous requests made by other services.

## Scaling based on Kafka message count

Kubernetes Cluster Scaling is configured with Terraform on Azure. We also have an HPA (Horizontal Pod Autoscaler) on the UserApprovalService deployment.

The HPA listens to a custom metric which provides information on how many messages are not yet processed in the Kafka topic user-approve. If there are messages queued up we spin up more pods.

The UserApprovalService sleeps 200 milliseconds after processing a message. This means it will lag behind if it’s the only instance and new messages coming in constantly.

## Monitoring & Metrics

We use Prometheus and Grafana to visualise what’s happening.

### Kafka metrics

To get metrics from Kafka we use the [kafka_exporter](https://github.com/danielqsj/kafka_exporter) which makes these available in Prometheus and by that Grafana. We implemented the kafka_exporter as sidecar in every Pod of the UserApprovalService so that the metrics can be used from/for every single Pod.

To get these Kafka Prometheus metrics usable as K8s custom metrics (required for HPA) we use the [k8s-prometheus-adapter](https://github.com/DirectXMan12/k8s-prometheus-adapter).

    # confirm install
    kubectl api-versions | grep "custom.metrics"

    # list Kafka topic metrics
    kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1  | jq

    # list Kafka topic metrics for every pod
    kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pod/*/kafka_consumergroup_lag" | jq

More details in the project’s [prometheus-adapter.yaml](https://github.com/wuestkamp/scalable-microservice-demo/blob/master/infrastructure/helm/config/prometheus-adapter.yaml). Now we’re able to use these Kafka metrics for K8s HPA.

### Istio metrics / Kiali

Kiali works fantastic with Istio and instantly provides us with an overview of what's happening:

![Image 6: Kiali network](https://cdn-images-1.medium.com/max/2280/1*51zyUCqAybYQrKvdAghBwQ.png)*Image 6: Kiali network*

In image 6 we see the REST requests hitting Istio Gateway and then OperationService. All other communication is going through “PassthroughCluster” which is the external managed Kafka. We can also see that the kafka-exporter is communicating with Kafka to gather metrics.

As of today, Istio is not able to manage Kafka traffic in more detail, it handles it as TCP. Envoy seems to be able to do this already, which means Istio will follow. We might then also see advances in Kiali, like displaying the number of messages per second on edges.

See more here in the Twitter thread of Joel Takvorian where he managed to include the Kafka nodes into the Kiali service diagram.

<iframe src="https://medium.com/media/384501a35399fbcd1e1eaa6dd78354a5" frameborder=0></iframe>

## In Action

Now the fun begins.

### UserApprovalService lagging behind

Without the HPA enabled, we create ~60 new events per second.

![Image 7: topic user-approve lag is rising](https://cdn-images-1.medium.com/max/4756/1*zDjRe4Hkp95iEPVCWfxzEg.png)*Image 7: topic user-approve lag is rising*

**From left to right we see:**

* Kafka events/second

* not yet handled (lagging) user-approve events

* amount UserApprovalService pods

* amount of nodes

The UserApprovalService sleeps 200 milliseconds after processing a message. This means it will lag behind if its only a single instance and new events coming in constantly, which we see in image 7.

### Enable scaling

Now with the HPA enabled and constantly rising incoming REST requests to create new users.

![Image 8: no load, but we start with 9 Nodes for faster scaling](https://cdn-images-1.medium.com/max/4768/1*XpIAl39Db6sUpYMY8WUonQ.png)*Image 8: no load, but we start with 9 Nodes for faster scaling*

![Image 9: Requests hitting, we see HPA scaling up the Kafka consumer UserApprovalService](https://cdn-images-1.medium.com/max/4780/1*1Pkz184hzRiYQ6PWKMYufA.png)*Image 9: Requests hitting, we see HPA scaling up the Kafka consumer UserApprovalService*

![Image 10: 2045 Events per second](https://cdn-images-1.medium.com/max/4772/1*PxDR-iQUUByDVULdmGdCrw.png)*Image 10: 2045 Events per second*

![Image 11: First node scaling](https://cdn-images-1.medium.com/max/4776/1*_rILSbTxUhQM0_RFdWDNoQ.png)*Image 11: First node scaling*

![Image 12: Close to 20000 Events per second](https://cdn-images-1.medium.com/max/4764/1*89a3v09ggt_K0Uqw-Bel1w.png)*Image 12: Close to 20000 Events per second*

![Image 13: 23000 !](https://cdn-images-1.medium.com/max/4772/1*38YDpWoGJaegBICt8hNy5Q.png)*Image 13: 23000 !*

## Recap

The app scaled up automatically to ~280 UserApprovalService pods, 11 Nodes and processed ~23000 Events per second! That’s a nice start :)

This was only stopped by the compute limitation of the Azure account and no more nodes could be created.

## Sources

* An amazing book which I enjoyed very much: Microservice Patterns by Chris Richardson. Much information from this book can also be found on [https://microservices.io](https://microservices.io)

* [A great article on how to use HPA with Kafka](https://medium.com/@ranrubin/horizontal-pod-autoscaling-hpa-triggered-by-kafka-event-f30fe99f3948)

## Become Kubernetes Certified

![[https://killer.sh](https://killer.sh)](https://cdn-images-1.medium.com/max/3534/1*7Kbj17_6VncUuoBqNsAzzg.png)*[https://killer.sh](https://killer.sh)*
