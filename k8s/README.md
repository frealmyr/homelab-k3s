# Kubernetes Applications

This folder contains applications which is monitored and deployed by ArgoCD.

  - Each application is part of a ArgoCD `application` using app-of-apps pattern.
  - Each ArgoCD `application` belongs to a argo `appProject`, for categorizing applications in the ArgoCD UI
  - Namespaces can either be shared, such as the `kube-system` namespace. Or they can be individually made for a single application stack, such as the `traefik-system` namespace.

ArgoCD configuration for the application stacks is defined in the `stack.yaml`, which is values for the `../chart/argo` helm chart.

> The argo helm chart is monitored by argocd, and is the first argo application that is installed during the initial bootstrapping of the cluster using Terraform.

## Why use this pattern instead of a folder structure with manifests?

My goal with the helm chart pattern is to reduce manifest boilerplate, at the cost of increased complexity.

If I would use a simple folder structure, I would need manifests for each top-level _application_ manifest which targets a folder, and in that folder I would need individually _application_ manifests for each _app-of-apps_ application.

Utilizing helm templates allows me to just create a folder with a helm chart and value, checking in the application as a key-value pair in `stack.yaml` and running sync in ArgoCD to deploy a new application.

That said, I might return to the simpler configuration sometime, as it breaks the ArgoCD Git webhook, requiring me to manually sync the applications.

And no, `syncPolicy.automated` is not a solution as it uses too much resources. Especially with Helm dependencies, as it basically ignores cache and downloads all depedencies to check the diff.

## New application flow

  - Create or re-use `appProject` folder
  - Create a new folder for the new _app-of-apps_ under the `appProject` folder
  - Add helm chart, kustomize or pure manifests to the application folder
  - Add entry in `stack.yaml` for the new application
    - Add `type:` key with value `kustomize` or `manifests` if the application does not contain `helm`.
    - Namespaces can specified with:
      - use existing namespace by specifying `namespace.name`
      - create a new namespace with the same name as application using `namespace.create=true`
      - combination of both `namespace.name` and `namespace.create=true` to create a new namespace with a name other than the application name.
  - Sync the `Homelab` application in the ArgoCD UI/CLI
  - Sync the new application after ArgoCD have detected the application
