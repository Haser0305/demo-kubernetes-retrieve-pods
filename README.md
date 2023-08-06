# Retrieve all pods 

The goal is to retrieve all pods in the same namespace, namely `test-namespace`. 
For this, we are creating few kubernetes objects.

## Service Account

This service account named `pod-reader-account` is created in the `test-namespace` namespace.

## Role
The role named pod-reader is created in the test-namespace namespace. This role has permissions to get, 
watch, and list pods.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: test-namespace
  name: pod-reader
rules:
  - apiGroups: [ "" ]
    resources: [ "pods" ]
    verbs: [ "get", "watch", "list" ]
```

## Role Binding
This RoleBinding binds the service account pod-reader-account to the role pod-reader, 
providing the permissions specified in the role to the service account.
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: test-namespace
subjects:
  - kind: ServiceAccount
    name: pod-reader-account
    namespace: test-namespace
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## Job
This Job named retrieve-all-pods-job uses the service account pod-reader-account 
to execute a curl command inside a pod. The curl command sends a request to the Kubernetes API Server 
to retrieve all pods in the test-namespace namespace.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: retrieve-all-pods-job
spec:
  template:
    spec:
      serviceAccountName: pod-reader-account
      containers:
        - name: curl
          image: curlimages/curl
          command:
            - "/bin/sh"
            - "-c"
          args:
            - >
              curl -k https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/api/v1/namespaces/test-namespace/pods
              -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"
      restartPolicy: Never
  backoffLimit: 1
```

This file provides an overview of how you are going to achieve 
the retrieval of all pods in the same namespace through ServiceAccount, 
Role, RoleBinding, and Job in Kubernetes.