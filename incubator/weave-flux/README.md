# Weave Flux

> ***NOTE: This chart is for Kubernetes version 1.6 and later.***

**TL;DR:** Weave Flux does GitOps for Kubernetes.

Weave Flux is an add-on to Kubernetes which implements release automation (continuous deployment) through [GitOps][].

[GitOps]: https://www.weave.works/blog/gitops-operations-by-pull-request

You install Weave Flux in your cluster and provide it access to a git repository where you store your application manifests, it will then ensure the manifests
are applied to the cluster as you make changes in git.

It can also automatically update manifests in git with image tags that it sees in your registry, and of course it would also ensure those manifests get applied
to the cluster straight away.

Weave Flux can optionally connect your cluster to Weave Cloud, which is a service that adds graphical user interface on top of open-source version of Weave Flux.
Weave Cloud also provides enterprise features for continuous delivery, and includes hosted Prometheus monitoring as well as other tools aimed to accelerate
delivery of cloud-native applications.

_To learn more about Weave Cloud and sign up for free please visit [Weaveworks website](https://weave.works)._

## Installing the Chart

To install this chart in standalone mode:

```console
$ helm install --name weave-flux --namespace kube-system --set Git.URL=<YOUR_CONFIG_REPO> incubator/weave-flux
```

You can also set `Git.Branch` and `Git.Path` see Configuration section below.

To upgrade to Weave Cloud mode, you will need a service token which you can get from [cloud.weave.works](https://cloud.weave.works/).

```console
$ helm upgrade --reuse-values --set WeaveCloud.ServiceToken=<YOUR_WEAVE_CLOUD_SERVICE_TOKEN> weave-flux incubator/weave-flux
```

## Usage Flux without Weave Cloud (standalone mode)

In standalone mode you will have to get `fluxctl` command, please download and install one from [the release page](https://github.com/weaveworks/flux/releases/latest).

You will need to forward agent port:
```console
$ kubectl port-forward -n kube-system "$(kubectl get -n kube-system pod -l weave-flux-component=agent -o jsonpath='{.items..metadata.name}')" 3030 &
```

You then should be able to use `fluxctl`, e.g.:
```console
$ fluxctl -u http://localhost:3030/api/flux list-services
```

If this works, you will need to ensure Weave Flux agent has access to the git repository.
Flux generates a deploy key for you, so you will need to obtain it:
```console
$ fluxctl -u http://localhost:3030/api/flux identity
```

Once you have the deploy key, you can add the key to your GitHub or GitLab repository, or any other git
hosting provider which supports deploy keys for access management.

See `fluxctl help` for more commands.

## Troubleshooting

To view pods created from resources in this chart:
```console
$ kubectl get -n kube-system pods -l weave-flux-component
```

To view the logs of the agent pod:
```console
$ kubectl logs -n kube-system "$(kubectl get -n kube-system pod -l weave-flux-component=agent -o jsonpath='{.items..metadata.name}')"
```

## Upgrading the Chart

To upgrade the chart to latest version:
```console
$ helm upgrade --reuse-values weave-flux incubator/weave-flux
```

## Uninstalling the Chart

To uninstall/delete the `weave-flux` chart:

```console
$ helm delete weave-flux
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following tables lists the configurable parameters of the chart and their default values.

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `WeaveCloud.ServiceToken` | Weave Cloud service token | _none_ _(**optional**)_ |
| `Git.URL` | URL of git repo with Kubernetes manifests; e.g., git@github.com:weaveworks/flux-example | _none_ _(**required**)_ |
| `Git.Branch` |  branch of git repo to use for Kubernetes manifests | `"master"` _(**optional**)_ |
| `Git.Path` | path within git repo to locate Kubernetes manifests (relative path) | `""` _(**optional**)_ |
