By default in kubernetes, every pod gets assigned an lcoal IP address.  
And every container in the pod gets assigned that same IP address. 
> The applications in a pod all use the same network namespace (same IP and port space), and can thus “find” each other and communicate using localhost. Because of this, applications in a pod must coordinate their usage of ports. Each pod has an IP address in a flat shared networking space that has full communication with other physical computers and pods across the network.

In ECP though you can request and assign an public IP address to your PODs. For that ECP provides a custom object `IPool`

### Create Reserved IP Pool
**You need permission to create IPool object in ECP**    

To Create a reserved IP pool define a `IPool` object in `ip-pool.yml` as:
```yml
apiVersion: v1 
kind: IPool 
metadata:
   name: ipool-example
   namespace: opseng 
spec:
   enableExternalIP: true # Optional. Set to “true” if static public IPV4 address is required 
   enableIpv6ExternalIP: false #   Optional. Set to “true” if static public IPV6 address is required
   counts: 3 # The count of static addresses needed. IPV4 and IPV6 addresses are counted separately.
```
To create the `IPool` 
```bash
kubectl create -f ip-pool.yml
```
or 
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   -v $(PWD)/ip-pool.yml:/ip-pool.yml \
   kubectl:2.2.2 create -f ip-pool.yml
```
Once a `IPool` created you will get 3 reserved IP's for your applcation.  
The IP's that are reserved for you will not change when you scale up/down. Those IP's gets released when the IPool object is deleted.  
A ipool object is always mapped to only one POD template. If your replication factor is more that the IP counts in reserved pool, it will cause an error. 



### Assign IP Pool to Pods
