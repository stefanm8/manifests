# Kubernetes Manifests

A collection of reusable Kubernetes manifest components designed to be included in other Kustomization files.

## Overview

This repository contains production-ready Kubernetes manifests for common data storage and caching solutions. Each component is packaged as a standalone YAML file that can be easily referenced and customized using Kustomize.

## Available Components

### MongoDB
**File:** `mongodb.yaml`

MongoDB StatefulSet with persistent storage.

- **Service:** Headless service on port 27017
- **Image:** `mongo:6.0`
- **Storage:** 1Gi persistent volume
- **Default Credentials:**
  - Username: `admin`
  - Password: `password`

### Redis
**File:** `redis.yaml`

Redis StatefulSet with AOF persistence enabled.

- **Service:** Headless service on port 6379
- **Image:** `redis:7-alpine`
- **Storage:** 1Gi persistent volume
- **Persistence:** AOF (Append Only File) enabled

### Dgraph
**File:** `dgraph.yaml`

Complete Dgraph graph database setup with Zero and Alpha nodes.

**Dgraph Zero:**
- Coordination service
- Ports: 5080 (gRPC), 6080 (HTTP)
- Health checks included

**Dgraph Alpha:**
- Data service with internal and public endpoints
- Ports: 7080 (HTTP), 8080 (gRPC), 9080 (internal gRPC)
- Health checks included
- Storage: 1Gi persistent volume per component

## Usage

### Using with Kustomize

Reference these manifests in your `kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - https://github.com/stefanm8/manifests/mongodb.yaml
  - https://github.com/stefanm8/manifests/redis.yaml
```

### Using with Local Path

If you've cloned this repository:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../manifests/mongodb.yaml
  - ../manifests/redis.yaml
```

### Customizing Resources

Override default values using Kustomize patches:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../manifests/mongodb.yaml

patches:
  - target:
      kind: StatefulSet
      name: mongodb
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 3
      - op: replace
        path: /spec/volumeClaimTemplates/0/spec/resources/requests/storage
        value: 10Gi
```

### Environment-Specific Secrets

Replace default credentials with Kubernetes secrets:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../manifests/mongodb.yaml

patches:
  - target:
      kind: StatefulSet
      name: mongodb
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/env
        value:
          - name: MONGO_INITDB_ROOT_USERNAME
            valueFrom:
              secretKeyRef:
                name: mongodb-credentials
                key: username
          - name: MONGO_INITDB_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mongodb-credentials
                key: password
```

## Deployment

Apply directly with kubectl:

```bash
kubectl apply -f mongodb.yaml
kubectl apply -f redis.yaml
kubectl apply -f dgraph.yaml
```

Or use with Kustomize:

```bash
kubectl apply -k .
```

## Storage Requirements

All components use `ReadWriteOnce` persistent volumes with 1Gi default storage:

- **MongoDB:** 1Gi for data directory (`/data/db`)
- **Redis:** 1Gi for data directory (`/data`)
- **Dgraph Zero:** 1Gi for coordination data
- **Dgraph Alpha:** 1Gi for graph data

Ensure your cluster has a default StorageClass configured or modify the volume claim templates to specify a StorageClass.

## Connection Strings

### MongoDB
```
mongodb://admin:password@mongodb:27017/
```

### Redis
```
redis://redis:6379
```

### Dgraph
```
# gRPC endpoint
dgraph-alpha-public:8080

# HTTP endpoint
dgraph-alpha-public:9080
```

## Security Considerations

**Important:** The default configurations include hardcoded credentials for development purposes. For production deployments:

1. Use Kubernetes Secrets for sensitive data
2. Enable authentication and TLS
3. Configure network policies
4. Use strong, randomly generated passwords
5. Consider using managed database services for production workloads

## Contributing

Feel free to submit issues or pull requests to improve these manifests.

## License

This project is provided as-is for use in your Kubernetes deployments.
