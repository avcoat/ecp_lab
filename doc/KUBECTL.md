ECP is based on Kubenets and ECP CLI is based on Kubernets CLI [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/). 
ECP provides a custom build of kubectl that can be used to deploy your application and define kubernets resources in ECP. 

Currently ECP only ships a rpm package. To install you need a linux distribution with rpm package manager.   
To install the rpm package 
```bash
rpm -ivh kubectl-2.2.2-1.el7.noarch.x86_64.rpm
```
  
Although if you dont have a linux distribution you can use the [dockerized version](https://github.com/avcoat/docker-kubectl).
```bash
docker pull registry-qcc.quantil.com/ibe-tools/kubectl:2.2.2 \
  && docker tag registry-qcc.quantil.com/ibe-tools/kubectl:2.2.2 kubectl:2.2.2
```
   
   
**Note: Offical kubectl will not work with ECP**
