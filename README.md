[![GitHub workflow status - Interactive Oceans Portal](https://img.shields.io/github/workflow/status/cormorack/helm-charts/Test%20Portal%20Chart?logo=github&label=Interactive%20Oceans%20Services)](https://github.com/cormorack/helm-charts/actions)

# Interactive Oceans Helm Charts

This repository contains the Interactive Oceans Services helm chart.

## Local Testing

0. Install docker, k3d, kubectl, helm, and stern if it doesn't exists in system yet

    ```bash
    # Installs docker in (Ubuntu)
    # Source: https://rancher.com/docs/rancher/v2.x/en/installation/requirements/installing-docker/
    curl https://releases.rancher.com/install-docker/19.03.sh | sh

    # Allow docker to run as non-root user
    sudo usermod -aG docker ubuntu

    # Installs k3d in Linux
    curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash

    # Install kubectl
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/

    # Install helm
    curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

    # Install stern
    wget -O stern https://github.com/wercker/stern/releases/download/$(curl -s https://api.github.com/repos/wercker/stern/releases/latest | grep tag_name | cut -d '"' -f 4)/stern_linux_amd64
    chmod +x ./stern
    sudo mv ./stern /usr/local/bin/
    ```

1. Spin up [k3s](https://k3s.io/) cluster via [k3d](https://k3d.io/). K3s is a lightweight kubernetes by [rancher](https://rancher.com/).

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

7. Clean up with the following commands

    ```bash
    helm delete io2-portal
    kubectl delete secrets cava-secrets
    ```

8. Tear down k3s cluster

    ```bash
    k3d delete
    ```
