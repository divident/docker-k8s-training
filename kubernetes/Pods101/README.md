## Pods101

 - [Introductory Slides](https://collabnix.github.io/kubelabs/Pods101_slides/Pods101.html) 
 - [Deploying Your First Nginx Pod](./deploy-your-first-nginx-pod.md) 
 - [Viewing Your Pod](./deploy-your-first-nginx-pod.md#viewing-your-pods) 
 - [Where is your Pod running on?](./deploy-your-first-nginx-pod.md#which-node-is-this-pod-running-on) 
 - [Pod Output in JSON](./deploy-your-first-nginx-pod.md#output-in-json) 
 - [Executing Commands against Pod](./deploy-your-first-nginx-pod.md#executing-commands-against-pods) 
 - [Terminating a Pod](./deploy-your-first-nginx-pod.md#deleting-the-pod) 
 - [Adding a 2nd container to a Pod](./deploy-your-first-nginx-pod.md#ading-a-2nd-container-to-a-pod) 
 - [Labels and Selectors in a Pod](./labels-and-selectors/README.md)

## Useful commands

List all available api-resources
```
kubectl api-resources 
```


Show schema of a resource
```
kubectl explain pod
kubectl explain --recursive pod
```
## Exercices
1. Create a pod with image nginx called nginx and expose traffic on port 80. Create second pod and curl index page of nginx pod on port 80.
1. Create 2 containers in a single pod. First container should write 'hello' every 2s to /var/logs/hello.txt file. You can use while loop 
```
while true; do
    echo "hello" >> /var/logs/hello.txt;
    sleep 2;
done
```
Second container should output log file contents using command `tail -f /var/logs/hello.txt`
