# Prometheus and Grafana

Prometheus and Grafana are commonly used for monitoring and visualization in Kubernetes environments, but they can also be used for monitoring other systems and services, including AWS SNS and other microservices. 

Prometheus is a versatile monitoring tool that can be configured to scrape metrics from a wide variety of sources, including AWS services and custom microservices. By setting up appropriate exporters and configurations, Prometheus can collect and store metrics from these services, allowing you to monitor their performance and health. 

Grafana, on the other hand, is a visualization tool that can be used to create dashboards and graphs based on the metrics collected by Prometheus. By connecting Grafana to Prometheus, you can create custom dashboards to monitor the performance of your AWS SNS and other microservices. 

In conclusion, while Prometheus and Grafana are commonly associated with Kubernetes, they can also be used to monitor and visualize a wide range of systems and services, including AWS SNS and other microservices.

Let's use see the comparison between fluentd, prometheus and grafana in term of observability.

| Feature       | Fluentd            | Prometheus         | Grafana            |
|---------------|--------------------|--------------------|--------------------|
| Data Collection | Collects logs and events from various sources | Collects metrics and time-series data | Visualizes data from various sources |
| Data Storage   | Stores data in a buffer before forwarding | Stores data in a time-series database | Does not store data, relies on external sources |
| Querying       | Supports querying and filtering of data | Supports querying and alerting based on metrics | Supports querying and visualization of data |
| Alerting       | Supports alerting based on log data | Supports alerting based on metrics | Supports alerting based on data visualization |
| Visualization  | Limited visualization capabilities | Limited visualization capabilities | Advanced visualization capabilities |
| Integration    | Integrates with various data sources and tools | Integrates with various monitoring tools | Integrates with various data sources and visualization tools |
| Scalability    | Scalable for handling large volumes of data | Scalable for handling large volumes of metrics | Scalable for handling large volumes of data visualization |
| Community      | Active community support and plugins | Active community support and exporters | Active community support and plugins |

![](/images/15_image01.png)

```sh
$ sudo snap install helm --classic
  helm 3.14.2 from Snapcrafters✪ installed

$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
  "prometheus-community" has been added to your repositories

$ helm repo update
  Hang tight while we grab the latest from your chart repositories...
  ...Successfully got an update from the "prometheus-community" chart repository
  Update Complete. ⎈Happy Helming!⎈

$ kubectl create namespace monitoring
  namespace/monitoring created

$ helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
  NAME: monitoring
  LAST DEPLOYED: Tue Mar 12 22:16:40 2024
  NAMESPACE: monitoring
  STATUS: deployed
  REVISION: 1
  NOTES:
  kube-prometheus-stack has been installed. Check its status by running:
    kubectl --namespace monitoring get pods -l "release=monitoring"

  Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.

```
To enable some visualization from Prometheus, we need some pods to play around with.<br>
We could do several way by using AWS EKS cluster or just simple using below yaml file from Docker Desktop.

```sh
$ cat nginx.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx
          ports:
          - containerPort: 80

$ kubectl apply -f nginx.yaml
  deployment.apps/nginx-deployment created

$ kubectl get pods
  NAME                                READY   STATUS              RESTARTS   AGE
  nginx-deployment-7c5ddbdf54-6rj99   0/1     ContainerCreating   0          5s
  nginx-deployment-7c5ddbdf54-kbbst   0/1     ContainerCreating   0          5s
  nginx-deployment-7c5ddbdf54-wtrpw   0/1     ContainerCreating   0          5s

$ kubectl port-forward service/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring
  Forwarding from 127.0.0.1:9090 -> 9090
  Forwarding from [::1]:9090 -> 9090
  Handling connection for 9090
  Handling connection for 9090
  Handling connection for 9090
```

In this case we are using http://127.0.0.1:9090/ from the browser and normally give us as follow.<br>
By picking prometheus_http_request_duration_seconds_count for example, we could see something like below.

![](/images/15_image02.png)

Similiarly we can do the same for Grafana as follow

```sh
kubectl port-forward service/monitoring-grafana 8080:80 -n monitoring
  Forwarding from 127.0.0.1:8080 -> 3000
  Forwarding from [::1]:8080 -> 3000
  Handling connection for 8080
  Handling connection for 8080
  Handling connection for 8080
  Handling connection for 8080
  Handling connection for 8080
```
By going to the browser http://127.0.0.1:8080/
and entering the default value below
![](/images/15_image03.png)
```sh
Default credentials
  User: admin
  Password: prom-operator
```
we should see something like below
![](/images/15_image04.png)

We may not see much as when we run EKS cluster, but with those 3 simple NGINX pods above <br>
we still could see Kubernetes / API server dashboard as follow
![](/images/15_image05.png)


Let's do a simple stree test like below<br>
To do so we need a docker pull like below 

```sh
$ docker pull containerstack/cpustress
  Using default tag: latest
  latest: Pulling from containerstack/cpustress
  c76b39ada3ed: Pull complete 
  badc43fe7dea: Pull complete 
  Digest: sha256:a72da3632d53fc69a5f60715a1b594b56e1a7d94c40963efbd4cdc37ca37d77e
  Status: Downloaded newer image for containerstack/cpustress:latest
  docker.io/containerstack/cpustress:latest

  What's Next?
    View a summary of image vulnerabilities and recommendations → docker scout quickview containerstack/cpustress
```
Then we can start putting a stress like below
```sh

$ kubectl run cpu-test --image=containerstack/cpustress -- --cpu 4 --timeout 90s --metrics-brief
  pod/cpu-test created

$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
cpu-test                            1/1     Running   0          15s
```

running the command 
```sh
kubectl run cpu-test --image=containerstack/cpustress --cpu 4 --timeout 90s --metrics-brief
``` 
will put a strain on your computer's CPU by running a stress test like couple spikes below.

![](/images/15_image06.png)

To stop the stress test, you can delete the pod that was created by running the command. You can do this by running the following command:

```
kubectl delete pod cpu-test
```
# SUMMARY
There are other good visualization package we can import for Grafana.
One of them is Node Exporter Full like below. See this [link](https://www.youtube.com/watch?v=yrscZ-kGc_Y) at minute 28.34.
![](/images/15_image07.png)

Other thing we don't really cover is PromQL. <br>
PromQL is the language that is used in Grafana and it visualize data from Prometheus as tables or plots.

![](/images/15_image08.png)