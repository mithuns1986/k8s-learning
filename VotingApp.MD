# End-to-End Voting Application on Kubernetes with Deployments and Services

## Namespace

Create an isolated namespace for all voting-app resources.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: vote
```

---

## Redis

Redis stores votes temporarily. We expose it internally via a ClusterIP service.

### Redis Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  namespace: vote
  labels:
    app: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:alpine
        ports:
        - containerPort: 6379
```

### Redis Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: vote
  labels:
    app: redis
spec:
  type: ClusterIP
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
```

---

## Voting Front-End

The Python-based voting UI accepts user votes. Exposed externally via NodePort.

### Voting Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-deployment
  namespace: vote
  labels:
    app: voting
spec:
  replicas: 3
  selector:
    matchLabels:
      app: voting
  template:
    metadata:
      labels:
        app: voting
    spec:
      containers:
      - name: voting
        image: kodekloud/examplevotingapp_vote:v1
        ports:
        - containerPort: 80
        env:
        - name: REDIS
          value: "redis-service"
```

### Voting Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: voting-service
  namespace: vote
  labels:
    app: voting
spec:
  type: NodePort
  selector:
    app: voting
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30001
```

---

## Worker

The background worker processes votes from Redis and writes them to PostgreSQL. No external Service required.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker-deployment
  namespace: vote
  labels:
    app: worker
spec:
  replicas: 1
  selector:
    matchLabels:
      app: worker
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers:
      - name: worker
        image: kodekloud/examplevotingapp_worker:v1
        env:
        - name: REDIS
          value: "redis-service"
        - name: DB
          value: "db-service"
```

---

## PostgreSQL

PostgreSQL provides persistent vote storage, exposed internally.

### DB Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-deployment
  namespace: vote
  labels:
    app: db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: db
        image: postgres:alpine
        env:
        - name: POSTGRES_USER
          value: "postgres"
        - name: POSTGRES_PASSWORD
          value: "postgres"
        - name: POSTGRES_DB
          value: "postgres"
        ports:
        - containerPort: 5432
```

### DB Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db-service
  namespace: vote
  labels:
    app: db
spec:
  type: ClusterIP
  selector:
    app: db
  ports:
  - port: 5432
    targetPort: 5432
```

---

## Result Front-End

The Node.js-based result UI fetches tallied votes from PostgreSQL. Exposed externally via NodePort.

### Result Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result-deployment
  namespace: vote
  labels:
    app: result
spec:
  replicas: 2
  selector:
    matchLabels:
      app: result
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
      - name: result
        image: kodekloud/examplevotingapp_result:v1
        ports:
        - containerPort: 80
        env:
        - name: DB
          value: "db-service"
```

### Result Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: result-service
  namespace: vote
  labels:
    app: result
spec:
  type: NodePort
  selector:
    app: result
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30002
```

---

## Applying Manifests

```bash
kubectl apply -f namespace.yaml
kubectl apply -n vote -f redis-deployment.yaml -f redis-service.yaml
kubectl apply -n vote -f voting-deployment.yaml -f voting-service.yaml
kubectl apply -n vote -f worker-deployment.yaml
kubectl apply -n vote -f db-deployment.yaml -f db-service.yaml
kubectl apply -n vote -f result-deployment.yaml -f result-service.yaml
```

You can then verify:

```bash
kubectl get all -n vote
```

Access the apps at:
- Voting UI: `http://<NodeIP>:30001`
- Result UI: `http://<NodeIP>:30002`

##All internal communication (Redis ↔ Voting/Worker, Worker/Result ↔ DB) uses services named `redis-service` and `db-service`, matching the hard-coded hostnames in the application code.
