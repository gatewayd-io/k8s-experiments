# k8s-experiments

Experimenting with k8s for scalable GatewayD deployment

## Resources

The following resources were used to create this project:

* [Kubernetes](https://kubernetes.io/)
* [MicroK8s](https://microk8s.io/)

## Setup

1. Install MicroK8s on your machine. See [here](https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s) for instructions. The last command in the instructions will print the token you need to access the Kubernetes dashboard. The IP address and port number for the dashboard can be found in the `kubernetes-dashboard` service.

    ```bash
    sudo snap install microk8s --classic
    sudo apt update && apt install -y ufw
    sudo ufw allow in on cni0 && sudo ufw allow out on cni0
    sudo ufw default allow routed
    ```

2. Add your user to the `microk8s` group.

    ```bash
    sudo usermod -a -G microk8s $USER
    sudo chown -f -R $USER ~/.kube
    newgrp microk8s
    ```

3. Enable the following MicroK8s addons:
    * dns
    * dashboard
    * storage

    ```bash
    microk8s enable dns dashboard storage
    ```

4. See the status of the MicroK8s cluster and optionally get the token for the Kubernetes dashboard.

    ```bash
    microk8s kubectl get all --all-namespaces
    token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
    microk8s kubectl -n kube-system describe secret $token
    ```

5. Follow the instructions in the [how to deploy Postgres on kubernetes](https://www.airplane.dev/blog/deploy-postgres-on-kubernetes) article to deploy a Postgres instance. You can use the files starting with `postgres-` in this repo as a starting point.

    ```bash
    microk8s kubectl apply -f postgres-config.yaml
    microk8s kubectl apply -f postgres-pvc-pv.yaml
    microk8s kubectl apply -f postgres-deployment.yaml
    microk8s kubectl apply -f postgres-service.yaml
    ```

6. Install GatewayD on your machine.

    ```bash
    microk8s kubectl apply -f gatewayd-config.yaml
    microk8s kubectl apply -f gatewayd-deployment.yaml
    microk8s kubectl apply -f gatewayd-service.yaml
    ```

7. Scale GatewayD to any number of replicas you want.

    ```bash
    microk8s kubectl scale deployment gatewayd --replicas=3
    ```

8. Use the following command to find the exposed port for the GatewayD service.

    ```bash
    microk8s kubectl get all
    ```

9. Use the following command to test the GatewayD service. Replace the port number with the one you found in the previous step.

    ```bash
    psql postgres://postgres:postgres@localhost:31221/postgres
    ```
