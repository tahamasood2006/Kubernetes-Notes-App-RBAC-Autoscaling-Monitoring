# Simple Notes App
This is 2 tier architecture based app where I have designed the k8-deployment files along with Role Based Access Control and structured Role Binding also created a monitoring dashboard using Grafana and collected metrics through prometheus.

## Requirements
1. Python 3.9
2. Node.js
3. React


## Installation
1. Clone the repository
```
git clone https://github.com/tahamasood2006/k8s-RBAC-withPrometheus-project.git
```

# For Monitoring using Grafana
Note : make sure you have helm installed in your system 

1. helm install prometheus-stackprometheus-community/kube-prometheus-stack --namespace <ENTER_YOUR_NAMESPACE>

we need some port-forwarding as well

1. kubectl port-forward svc/prometheus-stack-grafan 3000:80 -n <ENTER_YOUR_NAMESPACE_FORMONITORINH> --address=0.0.0.0

After this we can do the configurations through UI

## Nginx

Install Nginx reverse proxy to make this application available

`sudo apt-get update`
`sudo apt install nginx`# k8s-RBAC-withPrometheus-project
