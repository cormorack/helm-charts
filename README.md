[![GitHub workflow status - Interactive Oceans Portal](https://img.shields.io/github/workflow/status/cormorack/helm-charts/Test%20Portal%20Chart?logo=github&label=Interactive%20Oceans%20Services)](https://github.com/cormorack/helm-charts/actions)

# Interactive Oceans Helm Charts

This repository contains the Interactive Oceans Services helm chart.

## Local Testing

1. Spin up k3s cluster

    ``` bash
    # Note: https://github.com/rancher/k3d/issues/104#issuecomment-542184960
    k3d create --server-arg --no-deploy --server-arg traefik
    ```

2. Connect to cluster and check

    ```bash
    export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"
    kubectl cluster-info
    ```

3. Update `io2-portal` helm chart dependencies

    ```bash
    helm dependencies update ./io2-portal
    ```

4. Install secrets

    ```bash
    kubectl apply -f .ci-helpers/deployments/secrets/cava-secrets.yaml
    ```

5. Install `io2-portal` chart.

    ```bash
    helm install io2-portal ./io2-portal --values .ci-helpers/deployments/secrets/dev-test.yaml
    ```

6. Any changes can be updated using following command

    ```bash
    helm upgrade -f .ci-helpers/deployments/secrets/dev-test.yaml io2-portal ./io2-portal
    ```
