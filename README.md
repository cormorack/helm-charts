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

1. Spin up [k3s](https://k3s.io/) cluster via [k3d](https://k3d.io/). K3s is a lightweight kubernetes by [rancher](https://rancher.com/).

    ``` bash
    # Note: https://github.com/rancher/k3d/issues/104#issuecomment-542184960
    # -p options are optional ... it is used to port forward to the host machine
    # Pinned to K3S v1.20 only for now and traefik is disabled
    
    k3d cluster create \
        --image rancher/k3s:v1.20.15-k3s1 \
        -p "9000:9000@loadbalancer" \
        -p "443:443@loadbalancer" \
        -p "80:80@loadbalancer" \
        --k3s-arg="--disable=traefik@server:0"
    ```

2. Connect to cluster and check

    ```bash
    kubectl config use-context k3d-k3s-default
    kubectl cluster-info
    ```

3. Create cava secrets

    **Must be unlocked first**

    ```bash
    kubectl apply -f .ci-helpers/deployments/secrets/cava-secrets.yaml
    ```

4. Update `io2-portal` helm chart dependencies

    ```bash
    helm dependencies update ./io2-portal
    ```

5. Install/upgrade `io2-portal` chart.

    ```bash
    helm upgrade io2-portal ./io2-portal --install --cleanup-on-fail --values .ci-helpers/deployments/secrets/dev-test.yaml
    ```

6. Delete chart only

    ```bash
    helm delete io2-portal
    ```

7. Tear down k3s cluster

    ```bash
    k3d cluster delete
    ```
