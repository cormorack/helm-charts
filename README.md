[![Portal Chart Deployment](https://github.com/cormorack/helm-charts/actions/workflows/deploy-chart.yaml/badge.svg)](https://github.com/cormorack/helm-charts/actions/workflows/deploy-chart.yaml)

# Interactive Oceans Helm Charts

This repository contains the Interactive Oceans Services helm chart.

## Local/Development Deployment

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
    # -p options are optional ... it is used to port forward to the host machine
    
    k3d cluster create \
        -p "9000:9000@loadbalancer" \
        -p "443:443@loadbalancer" \
        -p "80:80@loadbalancer" \
        --k3s-server-arg \
        --no-deploy \
        --k3s-server-arg traefik
    ```

2. Connect to cluster and check

    ```bash
    kubectl config use-context k3d-k3s-default
    kubectl cluster-info
    ```

3. Update `io2-portal` helm chart dependencies

    ```bash
    helm dependencies update ./io2-portal
    ```

4. Install/upgrade `io2-portal` chart.

    ```bash
    helm upgrade io2-portal ./io2-portal --install --cleanup-on-fail --create-namespace --namespace development --values .ci-helpers/deployments/secrets/dev-test.yaml
    ```

5. Delete chart only

    ```bash
    helm delete --namespace development io2-portal
    ```

6. Tear down k3s cluster

    ```bash
    k3d cluster delete
    ```
