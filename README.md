# Blue/Green Test App for PR ApplicationSet

This is to test the [PR ApplicationSet Generator](https://argocd-applicationset.readthedocs.io/en/stable/Generators-Pull-Request/). This repo can be used while following [this blogpost](https://developers.redhat.com/articles/2022/04/05/automate-cicd-pull-requests-argo-cd-applicationsets#)

## Prereqs

You need a Kubernetes cluster. [Kind](kind.sigs.k8s.io/) is the easiest to test this.

```shell
kind create cluster
```

Install Argo CD.

```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

To access the Argo CD UI, first get the Admin password:

```shell
kubectl get secret/argocd-initial-admin-secret -n argocd -o jsonpath='{.data.password}' | base64 -d ; echo
```

In another window, do some port forwarding

```shell
kubectl -n argocd port-forward service/argocd-server 8080:443
```

Now you can visit the UI at [http://localhost:8080](http://localhost:8080)

## Testing

Deploy the  Application

```shell
kubectl apply -f https://raw.githubusercontent.com/christianh814/bgd/main/manifests/applications/bgd-app.yaml
```

Verify

```shell
$ kubectl get apps -n argocd
NAME   SYNC STATUS   HEALTH STATUS
bgd    Synced        Healthy
```

Deploy the ApplicationSet

```shell
kubectl apply -f https://raw.githubusercontent.com/christianh814/bgd/main/manifests/applicationsets/bgd-pr-appset.yaml
```

Verify that

```shell
$ kubectl get appsets -n argocd
NAME             AGE
bgd-pr-testing   51s
```

## Success

This should create an Application for [every PR listed
here](https://github.com/christianh814/bgd/pulls) that has the `preview`
label.

```shell
$ kubectl get apps -n argocd
NAME          SYNC STATUS   HEALTH STATUS
bgd           Synced        Healthy
bgd-green-3   Synced        Healthy
```

Note that out of the 2 PRs, only the one with the `preview` label is deployed.

Deleting namespaces
 - https://github.com/argoproj/argo-cd/issues/7875
 - https://github.com/argoproj/argo-cd/pull/11821 
