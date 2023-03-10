# Structure
### recources.yml
- This yaml  file contain resources to deploy, Which are as follow.
#### deployments.yml
- This file carry two deployments one for AMD64 and another for ARM64 base node.
- Container Port: `8080`
- Docker Image Name: `trinodb/trino` 
- Default Replicas: `1`
#### services.yml
- This yaml file carry two Loadbalancer services one for AMD64 and another for ARM64 base nodes deployments respectively.
- Default Listening Port : `8080`
- Can be ping on `loadbalancerdns:8080`
#### instance_groups.yml
-  This is a manifest yaml file for deploying instance groups with different CPU Architectures
>> Note: Please  make sure to use defined labels for NodeSelector in deployments.
# Setting Up Environment
### Packages required
##### kubectl
- curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.9/2023-01-11/bin/linux/amd64/kubectl
- chmod +x ./kubectl
- mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
##### kops
- curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
- chmod +x kops
- sudo mv kops /usr/local/bin/kops
## Setting Up Kops Cluster
>>Note: Make sure you have a private route53 hosted zone `example.com`
- aws s3api create-bucket --bucket kops-state-store-new --region us-west-2
- aws s3api put-bucket-versioning --bucket kops-state-store-new --versioning-configuration Status=Enabled
- export NAME=k8s-cluster.example.com
- export KOPS_STATE_STORE=s3://kops-state-store-new
- kops create cluster     --name=\${NAME}     --cloud=aws     --zones=us-west-2a,us-west-2b     --discovery-store=\${KOPS_STATE_STORE}/\${NAME}/discovery      --dns=private
- kops update cluster --name ${NAME} --yes --admin
## Setting Up Kops Nodes
- kops create -f instance_groups.yml --state=${KOPS_STATE_STORE}
- kops update cluster --name ${NAME} --yes
#### Deploy Command
- kubectl apply -k .

-----------------------------------------------------------------------------------------------------------
kops delete cluster --name=k8s-cluster-shadab.groveops.net   --state=s3://kops-state-store-new   --cloud=aws    --zones=us-west-2a,us-west-2b   --node-count=0   --dns=public
kops create -f arm-instance-group.yaml
kops update cluster --name k8s-cluster-shadab.groveops.net --yes --admin
export NAME=k8s-cluster.example.com
export KOPS_STATE_STORE=s3://kops-state-store-new
kubectl get nodes --show-labels
kubectl create namespace arm-namespace
kubectl apply -f arm-deployments.yaml -n arm-namespace
kops delete instance i-03777cbb12ff025ab --yes
kops export kubeconfig --admin

kubectl run -it --rm debug --image=busybox --restart=Never --namespace=default -- nslookup trino-coordinator.arm-namespace.svc.cluster.local

service_name.namespace.svc.cluster.local
my_trino.default.svc.cluster.local
