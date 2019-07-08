kubectl uses the configuration to communicate with different kubernets clusters. 
A sample config file looks like

```yaml
apiVersion: v1

  clusters:
  - name: minikube
    cluster:
      certificate-authority: /Users/swarvanusengupta/.minikube/ca.crt
      server: https://192.168.99.100:8443
    
      
  users:
  - name: minikube
    user:
      client-certificate: /Users/swarvanusengupta/.minikube/apiserver.crt
      client-key: /Users/swarvanusengupta/.minikube/apiserver.key  
    
  contexts:
  - name: minikube
    context:
      cluster: minikube
      user: minikube
    
    
  current-context: minikube
  kind: Config
  preferences: {}
```
The config mainly defines 
> 1. Clusters
> 2. Users
> 3. Context (Cluster + User)

Using kubectl you can switch between multiple contexts. 
```bash
kubectl config use-context minikube
```

If you doesn't operate on any other kubernets cluster, You can just replace the configuratiion with ecp config file   

Create a config file `config`
```yaml
apiVersion: v1

clusters:
- name: icn0
  cluster:
    insecure-skip-tls-verify: true
    server: https://qcc-api-icn0.quantil.com:443
  
  
users:
- name: opseng
  user:
    client-certificate: opseng_cert.pem
    client-key: opseng_key.pem

contexts:
- name: qcc
  context:
    cluster: icn0
    namespace: opseng
    user: opseng
  
  
current-context: context
kind: Config
preferences: {}
```
   
Overwrite the config
```
mv <config> $HOME/.kube/config 
```

Otherwise you can append the ecp config into the existing configuration `$HOME/.kube/config` as:
```yaml
apiVersion: v1
clusters:
- name: minikube
  cluster:
    certificate-authority: /Users/swarvanusengupta/.minikube/ca.crt
    server: https://192.168.99.100:8443
- name: icn0
  cluster:
    insecure-skip-tls-verify: true
    server: https://qcc-api-icn0.quantil.com:443

users:
- name: minikube
  user:
    client-certificate: /Users/swarvanusengupta/.minikube/apiserver.crt
    client-key: /Users/swarvanusengupta/.minikube/apiserver.key
- name: opseng
  user:
    client-certificate: opseng_cert.pem
    client-key: opseng_key.pem

contexts:
- name: minikube
  context:
    cluster: minikube
    user: minikube
- name: qcc
  context:
    cluster: icn0
    namespace: opseng
    user: opseng

current-context: qcc
kind: Config
preferences: {}
```
   
Copy your cert file and key file to `$HOME/.kube/`
```bash
mv <cert_file> $HOME/.kube/opseng_cert.pem
mv <key_file> $HOME/.kube/opseng_key.pem
```

To check the configuration perform
```bash
kubectl get pods
```

To run in docker
```bash
export CERT=$HOME/.kube/opseng_cert.pem
export PEM=$HOME/.kube/opseng_key.pem

docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \ 
   kubectl:2.2.2 get pods
```
