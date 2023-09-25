# Kubernetes Security

As individual developers, security is probably not something that is at the forefront of our minds. We focus mostly on fulfilling the requirements of a task and leave things like access control to be handled later. When it comes to working with Kubernetes, we tend to take the security aspect of things for granted. Since Kubernetes is built with security in mind, and since the authors of any emerging technologies need to give security its due care for the technology to be considered for commercial use, the Kubernetes ecosystem is relatively secure. However, when it comes to large organizations that set up large-scale clusters for internal and customer use, it is very important to have a significant amount of security measures in place. Most large organizations have entire teams dedicated to handling authorization and application security. Kubernetes depends on microservices, which use thousands of third-party applications to function. Each of these applications could potentially introduce a security vulnerability, and that is simply not acceptable. A breach that occurs for a client due to mishandled credentials could cost greatly for a business, and it is therefore in an organization's interest to ensure that they hire DevOps engineers that know security best practices. So let's take a deep dive into Kubernetes security.

## Securing images

When using third-party images, there is a possibility that the author of the image may not have taken the necessary steps to secure the image. Therefore, ensuring that images comply with the security standard set by your organization is up to you. However, if you are creating your image to be used internally or to be freely available on an image hosting site, there are several steps you should take to secure your image. After all, it would be rather embarrassing if an attacker managed to gain access to a cluster by exploiting a container running your image. So what can be done here? 

### Check your dependencies

Unless the image you are building only does a simple task, you would likely use a base image or a list of images that your image depends on. You would then build your image on top of this base image. This is one place where things can go wrong since you have little control over the base images and their content. Additionally, you may have images that interact with the underlying operating system and performs some commands that allow attackers to see a backdoor in the system. The easiest way to circumvent this issue is to use as few dependencies as possible. The more images you depend on, the greater the chance of a security risk. When choosing a dependency, make sure you only get an image that has exactly what you want. For instance, if you want to curl, then there is no reason to choose a general-purpose image that has curl/wget/other request-handling commands instead of just getting an image that provides curl.

**Image scanning**: You might think that all the above steps sound complicated, but they shouldn't be, because image scanning exists. Image scanning allows you to automatically look at a database of regularly updated vulnerabilities and compare your image and its dependencies against it. Normally, you wouldn't build a commercial-grade image by hand, and would instead allow a pipeline to do that for you. You could simply add image scanning as an additional step that runs after the image itself has been built.

But vulnerabilities can be added after the image has been built, and if your image depends on other images, vulnerabilities can be introduced via those other images as well. The result of this is that you can't afford to scan your image once, push it into the container repo, and forget about it. You need to ensure that all the images that already exist in your registry are periodically scanned. You might have some help in this regard depending on the image registry you choose. For instance, registries such as GCR, AWS ECR, Docker hub, etc... have inbuilt repository image scanning capabilities. However, if you host your container registry, then you might need to do this manually.

A good example of an image scanning service is [Snyk](https://snyk.io). It's extremely simple to use, and you only need to run a single command:

```
docker scan <image-name>
```

Running this command on your release pipeline should flush out any vulnerabilities in your image. Another excellent tool that can be used is [Sysdig](https://sysdig.com). They provide [container security](https://sysdig.com/use-cases/container-security/) in the same way that Snyk does, allowing you to smoothly integrate image scanning into your existing pipeline. You also get continuous compliance which ensures that a new vulnerability that affects your image is not found after you release. This includes checking your configuration, ensuring that any credentials used within your image have not been leaked, and protecting your image against insider threats. You also get image compliance, allowing you to present proof that your image complies with any standards put in place by your organization. It also integrates smoothly with your existing [kubernetes clusters](https://sysdig.com/use-cases/kubernetes-security/), as well as your [IaC platforms](https://sysdig.com/products/secure/infrastructure-as-code-security/).

### User Service users

If you run your containers with users that have unrestricted access (such as a root user), then an attacker who gains access to your container can easily gain access to the host system since they already have elevated privileges. The solution to this problem is to create a service user when creating the container, and then to ensure that the container is handled by that un-privileged service user.  This way, even if an attacker gets access to the container, they won't be able to do much with the service account and would have to also get access to the root user before they can accomplish anything.

So how can you handle this? Docker runs commands as root by default, and you need to change this by adding the service user to your user group in the docker file. The ```usermod``` command can do this for you if you already have an existing user group, or you can create/add to user groups with:

```
RUN groupadd -r <appname> && useradd -g <appname> <appname>
```

The next step is to limit what the user can do within this image using the ```chown``` command:

```
RUN chown -R <appname>:<appname> /app
```

Next, switch to this user so that the container always runs with the user as opposed to the root user:

```
USER <appname>
```

Now, your container will run with the user you specified here, and it adds a layer of protection. However, when you spin up a pod using this image, you have to ensure that you do not misconfigure the yaml so that you override the image commands and start running your pod as root. To ensure this, place the commands:

```
allowPrivilegeEscalation: false
```

Read more about this [here](https://kubernetes.io/docs/concepts/security/pod-security-policy/#privilege-escalation).

### Maintain tight user groups and permissions

All the above precautions need to be taken when creating the image, well before it is deployed. However, these are just the first steps. Once you go ahead and deploy your images and have pods spun up from them, there are further actions that you need to take to prevent attackers from hacking into your system. It is a common conception that the users of a system are its weakest link, and while users need to be able to use the system in a way that doesn't allow it to be exploited, your job is to assume that an exploit will happen and to prepare accordingly. Note that your cluster may be accessed by accounts belonging to real people as well as automated system/service accounts. All these accounts need to be secured. If you give every single user in the organization admin privileges, then if just one of those accounts were to be compromised, your entire system will be at risk. Alternatively, if each user is grouped to a very specific set of permissions that allow the user to do only what they need to, and nothing more, then even if an account is compromised, the damage would be limited.

Luckily, Kubernetes has a solution in place so that you don't have to spend a significant amount of time setting things up. RBAC (Role-Based Access Control) allows you to specify roles, which in turn specify access.  So if you want to specify read access to a cluster's logs, but only logs about a specific namespace, you can create a role for that. Once you have a role in place, you can assign this role to any user(s), which will automatically give them all the privileges granted by that role. So this means if you have a team of people who use a specific set of permissions, you can create a role that has that set of permissions, and assign the roles to each member (or group members in a group and assign roles to the group) without having to assign each permission individually. As with everything else, you define RBAC as a Kubernetes resource that follows the standard Kubernetes Yaml format. A comprehensive look at RBAC can be found in the [RBAC101 section](../RBAC101/README.md)

### In-cluster network policies

Now that you have secured against any external attacks into your cluster, you can go ahead and assume that any security measures there will fail, and an attacker would eventually manage to find their way into the cluster. This means that you now have to ensure that your cluster is protected from the inside as well. 

By default, all the pods in a cluster are connected in some form. Most of them can access each other via localhost, and this means that if someone were to get into your cluster and find their way into one pod, every other pod would also be free for the taking. The solution to this is the same as the solution we came up with for user access: limited access. If you were to go ahead and restrict each pod's ability to communicate, this would solve the issue to some extent. A network policy would, as always, be defined as a Kubernetes resource of type ```NetworkPolicy```, and would allow you to specify ranges where of IPs that are acceptable to each resource.

While network policies are great for restricting access when it comes to a small cluster with a small number of pods, you might have some repetition and complexity when it comes to a larger cluster. Consider using a service mesh such as Istio and Linkerd to enhance the security (as well as add several other cluster-wide features) by automatically adding proxies to each pod that individually manages pod communication. You can learn more about this in the [ServiceMesh101 section](../ServiceMesh101/what-are-service-meshes.MD).

### Encryption

Despite all the above measures, pods still need to communicate. After all, that is the whole point of a microservice architecture. This is where encryption comes in. By default, any communication happening between pods happens unencrypted. This means that even if an attacker can't directly access the pods, they can gain access to the pod logs of every other pod. Using mutual TLS helps to solve the problem by encrypting inter-pod traffic. This is also available as part of Istio, which is why using service meshes in large clusters is always recommended.
