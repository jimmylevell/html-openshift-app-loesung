# ZLI-Modul-109-html-openshift-app (Lösung 6.2)

## Technologies

- Docker
- NGINX
- HTML
- OpenShift

---

## How To Use

### Build and push new Image with nginxinc/nginx-unprivileged

```bash
FROM nginxinc/nginx-unprivileged
# Copy the entire directory into the default directory of NGINX.
COPY /app /usr/share/nginx/html
```

```bash
docker build -f Dockerfile.web -t ghcr.io/jimmylevell/html-openshift-app:latest .
```

```bash
docker run -d -p 8080:8080 --name html-openshift-app ghcr.io/jimmylevell/html-openshift-app:latest
```

```bash
docker push ghcr.io/jimmylevell/html-openshift-app:latest
```


### OpenShift

#### 01-with-deployment

1. Sart

```bash
oc apply -f oc/01-with-deployment --recursive
```

2. Delete

```bash
oc delete all --selector=app=html-openshift-app
```

#### 02-with-template

1. Create OC template

```bash
oc apply -f oc/02-with-template --recursive
```

2. Start

```bash
oc process html-openshift-app \
  -p NAMESPACE="<Your-Namespace>" \
  -p GIT_SOURCE_URI="<Your-Git-Repo-URL>" \
  -p GIT_USERNAME="<Your-Git-Username>" \
  -p GIT_PAT="<Your-Git-PAT>" | oc apply -f -
```

3. Triggers

```bash
oc set triggers deploy/html-openshift-app --from-image=html-openshift-app:latest -c html-openshift-app
```

4. Delete

```bash
oc delete all --selector=app=html-openshift-app
```

5. Delete OC template

```bash
oc delete template html-openshift-app
```

#### 03-with-buildconfig

1. Andjust Namespace in Image URL of 030-Deployment.yaml

```bash
spec:
  template:
    spec:
      containers:
        - name: html-openshift-app
          image: image-registry.openshift-image-registry.svc:5000/<NAMESPACE>/html-openshift-app:latest
```

2. Start

```bash
oc apply -f oc/03-with-buildconfig --recursive
```

3. Triggers

```bash
oc set triggers deploy/html-openshift-app --from-image=html-openshift-app:latest -c html-openshift-app
```

4. Delete

```bash
oc delete all --selector=app=html-openshift-app
```

## Visualization
```mermaid
graph TB
    subgraph "External Access"
        Client[External Client/Browser]
    end

    subgraph "OpenShift Cluster"
        Route[Route<br/>html-openshift-app<br/>TLS: edge termination]

        Service[Service<br/>html-openshift-app<br/>Port: 8080]

        subgraph "Deployment"
            Deploy[Deployment<br/>html-openshift-app<br/>Replicas: 1]

            subgraph "Pod"
                Container[Container<br/>html-openshift-app<br/>Image: ghcr.io/modul-i-ch-109/html-openshift-app:v1<br/>Port: 8080]
            end
        end
    end

    Client -->|HTTPS| Route
    Route -->|HTTP| Service
    Service -->|Port 8080| Container
    Deploy -.manages.-> Container

    style Client fill:#e1f5ff
    style Route fill:#fff4e6
    style Service fill:#f3e5f5
    style Deploy fill:#e8f5e9
    style Container fill:#fce4ec
```
