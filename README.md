# Kubernetes Mutating Webhook for Resource Name Mutation

[![GoDoc](https://godoc.org/github.com/morvencao/kube-mutating-webhook-tutorial?status.svg)](https://godoc.org/github.com/morvencao/kube-mutating-webhook-tutorial)

This tutoral shows how to build and deploy a [MutatingAdmissionWebhook](https://kubernetes.io/docs/admin/admission-controllers/#mutatingadmissionwebhook-beta-in-19) that changes the name of a resource to the sha256 hash of its spec.

## Prerequisites

- [git](https://git-scm.com/downloads)
- [go](https://golang.org/dl/) version v1.12+
- [docker](https://docs.docker.com/install/) version 17.03+
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) version v1.11.3+
- Access to a Kubernetes v1.11.3+ cluster with the `admissionregistration.k8s.io/v1beta1` API enabled. Verify that by the following command:

```
kubectl api-versions | grep admissionregistration.k8s.io
```
The result should be:
```
admissionregistration.k8s.io/v1
admissionregistration.k8s.io/v1beta1
```

> Note: In addition, the `MutatingAdmissionWebhook` and `ValidatingAdmissionWebhook` admission controllers should be added and listed in the correct order in the admission-control flag of kube-apiserver.

## Build

1. Build binary

```
# make build
```

2. Build docker image
   
```
# make build-image
```

3. push docker image

```
# make push-image
```

> Note: log into the docker registry before pushing the image.

## Deploy

1. Create namespace `name-mutator` in which the name mutator webhook is deployed:

```
# kubectl create ns name-mutator
```

2. Create a signed cert/key pair and store it in a Kubernetes `secret` that will be consumed by name mutator deployment:

```
# ./deployment/webhook-create-signed-cert.sh \
    --service name-mutator-webhook-svc \
    --secret name-mutator-webhook-certs \
    --namespace name-mutator
```

3. Patch the `MutatingWebhookConfiguration` by set `caBundle` with correct value from Kubernetes cluster:

```
# cat deployment/mutatingwebhook.yaml | \
    deployment/webhook-patch-ca-bundle.sh > \
    deployment/mutatingwebhook-ca-bundle.yaml
```

4. Deploy resources:

```
# kubectl create -f deployment/deployment.yaml
# kubectl create -f deployment/service.yaml
# kubectl create -f deployment/mutatingwebhook-ca-bundle.yaml
```

## Verify

1. The name mutator webhook should be in running state:

```
# kubectl -n name-mutator get pod
NAME                                                   READY   STATUS    RESTARTS   AGE
name-mutator-webhook-deployment-7c8bc5f4c9-28c84       1/1     Running   0          30s
# kubectl -n name-mutator get deploy
NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
name-mutator-webhook-deployment       1/1     1            1           67s
```

2. Create new namespace `mutator` and label it with `name-mutation=enabled`:

```
# kubectl create ns mutator
# kubectl label namespace mutator name-mutation=enabled
# kubectl get namespace -L name-mutation
NAME                 STATUS   AGE   NAME-MUTATION
default              Active   26m
mutator              Active   13s   enabled
kube-public          Active   26m
kube-system          Active   26m
name-mutator         Active   17m
```

3. Deploy an app in Kubernetes cluster, take `alpine` app as an example

```
# kubectl run alpine --image=alpine --restart=Never -n mutator --command -- sleep infinity
```

4. Verify the name is mutated:

```
# kubectl get pod -n mutator
NAME                                                               READY   STATUS    RESTARTS   AGE
2a765e5498d15fb4b6aeb0e057f1645616bbc2f62e38e612a992243a10cf758f   1/1     Running   0          1m
```

## Troubleshooting

Check the following items:

1. The name-mutator webhook is in running state and no error logs.
2. The namespace in which application pod is deployed has the correct labels as configured in `mutatingwebhookconfiguration`.
3. Check the `caBundle` is patched to `mutatingwebhookconfiguration` object by checking if `caBundle` fields is empty.
