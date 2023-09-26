2. Final configuration should look like
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: wonderful
  name: wonderful-v2
spec:
  replicas: 4
  selector:
    matchLabels:
      app: wonderful
      version: v2
  template:
    metadata:
      labels:
        app: wonderful
        version: v2
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: wonderful
  name: wonderful
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30080
  selector:
    app: wonderful
    version: v2 # change
  type: NodePort
```
3. 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: wonderful
  name: wonderful-v1
spec:
  replicas: 8
  selector:
    matchLabels:
      app: wonderful
  template:
    metadata:
      labels:
        app: wonderful
    spec:
      containers:
      - image: httpd:alpine
        name: httpd
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: wonderful
  name: wonderful-v2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wonderful
  template:
    metadata:
      labels:
        app: wonderful
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
```