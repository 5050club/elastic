# elastic

# k3d setup

If the k3d cluster already exists w/o ports added for es stuff
> k3d node edit k3d-5050club-serverlb --port-add 5601:5601 --port-add 9200:9200

Run following command
> export KUBECONFIG="$(k3d kubeconfig write 5050club)"

Should be able to access es on 9200 and kb on 5601 from localhost once the cluster is spun up


# ECK
## setup ECK operator
https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-eck.html

> kubectl create -f https://download.elastic.co/downloads/eck/2.14.0/crds.yaml
> kca -f https://download.elastic.co/downloads/eck/2.14.0/operator.yaml

## create es and kb cluster

### create namespace and then es/kb cluster in that namespace
> kca -f /Users/j10s/apps/5050club/kubernetes/namespaces/elastic-clusters.yaml
> kubens elastic-clusters
> kca -f /Users/j10s/apps/5050club/elastic/k8s/es_simple_with_lb.yaml

A few commands to monitor/troubleshoot es cluster to make sure its healthy 
> kcg elasticsearch
> kc logs pod/es-5050club-es-default-0

Create Kibana
> kca -f /Users/j10s/apps/5050club/elastic/k8s/kibana_simple_with_lb.yaml


### authentication
A default user named elastic is created by default w/ pw stored in k8s secret

> PASSWORD=$(kubectl get secret es-5050club-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')
> echo $PASSWORD

# networking - es

To validate access to es, from cli:
> curl -u "elastic:$PASSWORD" -k "https://localhost:9200"

To validate access to kb, in a browser go to:
> https://localhost:5601


## Quick Alternative

This should only be done if other steps havent been followed to allow access (k3d w/ lb ports and cluster deployment w/o http spec).  This is just using a very simple elastic cluster deployment config w/o the http spec and port-forwarding for some temporary access.

A ClusterIP Service is auto created for es and kb
> kcg service es-5050club-es-http
> kcg service kb-5050club-kb-http

set up port forward for es and kb to connect from laptop
> kubectl port-forward service/es-5050club-es-http 9200
> kubectl port-forward service/kb-5050club-kb-http 5601

Get elastic password
> PASSWORD=$(kubectl get secret es-5050club-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')

To validate access to es, from cli:
> curl -u "elastic:$PASSWORD" -k "https://localhost:9200"

To validate access to kb, in a browser go to:
> https://localhost:5601

