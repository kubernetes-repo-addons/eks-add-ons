# EKS  Add-ons

Welcome to the EKS  Add-ons repository.

This repository contains GitOps configuration which follows the ArgoCD App of Apps pattern. It demonstrates how EKS  can leverage ArgoCD to easily bootstrap an EKS cluster with a wide variety of Kubernetes add-ons.

## Usage



### With an Existing Cluster

To bootstrap an existing cluster, you must first install ArgoCD. Instructions for doing so can be found in the ArgoCD [getting started documentation](https://argo-cd.readthedocs.io/en/stable/getting_started/).

#### Configure an Application

Once ArgoCD is installed, create a new `application.yaml` file with the following configuration.

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: add-ons
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/kubernetes-repo-addons/eks-add-ons.git
    targetRevision: HEAD
    path: chart
    helm:
      release: add-ons
      values: |
        cluster-autoscaler:
          enable: true
        metrics-server:
          enable: true
  destination:
    server: {{ .Values.destinationServer | default "https://kubernetes.default.svc" }}
    namespace: argocd
```

Note that you specify which add-ons should be installed by supplying values in the `spec.source.helm.values` map configuration. This example will only install `cluster-autoscaler` and `metrics-server`. To view available add-ons, visit the `add-ons` directory.

#### Deploy the add-ons

Apply the yaml to bootstrap the cluster.

```
kubectl apply -n argocd -f application.yaml
```

## Repo Structure

The structure of this repository follows the ArgoCD [App of Apps](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#app-of-apps) pattern. The configuration in this repository is organized into two directories: `chart` and `add-ons`.

```
├── chart
└── add-ons
```

### Chart

The `chart` directory contains a Helm chart which represents the root Application for the ArgoCD App of Apps pattern. This Helm chart in turn deploys additional ArgoCD Application resources which represent additional add-on Helm charts.

```
chart
├── templates
│   └── agones.yaml
│   └── appmesh-controller.yaml
│   └── calico.yaml
│   └── aws-cloudwatch-metrics-calico.yaml
│   └── aws-for-fluent-bit.yaml
│   └── aws-load-balancer-controller.yaml
│   └── aws-otel-collector.yaml
│   └── aws-cert-manager.yaml
│   └── ...
├── Chart.yaml
├── values.yaml
```

### Add-ons

The `add-ons` directory contains a dedicated Helm chart that deploys each individual add-on.

```
add-ons
├── agones
│   └── Chart.yaml
│   └── values.yaml
├── appmesh-controller
│   └── Chart.yaml
│   └── values.yaml
├── calico.yaml
│   └── Chart.yaml
├── aws-cloudwatch-metrics.yaml
│   └── Chart.yaml
│   └── values.yaml
│   └── ...
```


## License

This library is licensed under the MIT-0 License. See the LICENSE file.
