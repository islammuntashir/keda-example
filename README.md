A very simple demo to to show how keda works

Prepare Kafka Cluster

#For simplicity we will Install the latest version of strimzi kafka distribution. There are other installation methods but the following will suffice for experimentation.

### Create Namespace
kubectl create ns kafka

kubectl apply -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka

### Deploy the Kafka cluster. The following is an example from strimzi for a kafka cluster with a persistent storage.
kubectl apply -f https://strimzi.io/examples/latest/kafka/kafka-persistent-single.yaml -n kafka 


# Deploy the Kafka topic.
kubectl apply -f workload/deploy/kafka-topic.yaml

### Install KEDA
#I have used old version of keda 1.4.0 . I also include new version of deployment in keda folder. Consumer scaller will change depending on this version. Please check keda website
kubectl apply -f https://raw.githubusercontent.com/kedacore/keda/v1.4.1/deploy/crds/keda.k8s.io_scaledobjects_crd.yaml

kubectl apply -f https://raw.githubusercontent.com/kedacore/keda/v1.4.1/deploy/crds/keda.k8s.io_triggerauthentications_crd.yaml

kubectl apply -f https://raw.githubusercontent.com/kedacore/keda/v1.4.1/deploy/00-namespace.yaml

kubectl apply -f https://raw.githubusercontent.com/kedacore/keda/v1.4.1/deploy/01-service_account.yaml

kubectl apply -f https://raw.githubusercontent.com/kedacore/keda/v1.4.1/deploy/10-cluster_role.yaml

kubectl apply -f https://raw.githubusercontent.com/kedacore/keda/v1.4.1/deploy/11-role_binding.yaml

kubectl apply -f https://raw.githubusercontent.com/kedacore/keda/v1.4.1/deploy/12-operator.yaml

kubectl apply -f https://raw.githubusercontent.com/kedacore/keda/v1.4.1/deploy/20-metrics-cluster_role.yaml

kubectl apply -f https://raw.githubusercontent.com/kedacore/keda/v1.4.1/deploy/21-metrics-role_binding.yaml

kubectl apply -f https://raw.githubusercontent.com/kedacore/keda/v1.4.1/deploy/22-metrics-deployment.yaml

kubectl apply -f https://raw.githubusercontent.com/kedacore/keda/v1.4.1/deploy/23-metrics-service.yaml

kubectl apply -f https://raw.githubusercontent.com/kedacore/keda/v1.4.1/deploy/24-metrics-api_service.yaml

### Deploy Consumer
### Create the keda-sample namespace. 
kubectl create ns keda-sample

### Build and deploy the application.
docker build -f Dockerfile -t simple-consumer .

kubectl apply -f deploy/consumer-deployment.yml -n keda-sample
kubectl apply -f deploy/consumer-service.yml -n keda-sample

Please change image in consumer-deployment.yml file before deployment

### Deploy the Kafka scaler.
kubectl apply -f deploy/consumer-scaler.yml


### Generate events in kafka Topic

#### Generate event manually 

kubectl -n kafka run kafka-producer -ti --image=strimzi/kafka:0.18.0-kafka-2.5.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic messages

#### Generate Multiple event together

kubectl -n kafka run kafka-producer -ti --image=strimzi/kafka:0.18.0-kafka-2.5.0 --rm=true --restart=Never -- bin/kafka-producer-perf-test.sh --topic messages --throughput 3 --num-records 100 --record-size 4 --producer-props bootstrap.servers=my-cluster-kafka-bootstrap:9092

