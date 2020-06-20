<p align="center">
<img src="src/frontend/static/icons/Hipster_HeroLogoCyan.svg" width="300"/>
</p>
Demo: Deploy and Secure Micorservices Application on Kubernetes with Istio and Amabssador


# The Application:

**Online Boutique** is a cloud-native microservices demo application.
Online Boutique consists of a 10-tier microservices application. The application is a
web-based e-commerce app where users can browse items,
add them to the cart, and purchase them.

** This Demo Application is provided by the Google Cloud team, and we will it to demonstrate use of technologies like
Kubernetes/GKE, Istio, Ambassador Edge Stack,..**. This application
works on any Kubernetes cluster (such as a local one), as well as Google
Kubernetes Engine. It’s **easy to deploy with little to no configuration**.

## Screenshots

| Home Page                                                                                                         | Checkout Screen                                                                                                    |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [![Screenshot of store homepage](./docs/img/online-boutique-frontend-1.png)](./docs/img/online-boutique-frontend-1.png) | [![Screenshot of checkout screen](./docs/img/online-boutique-frontend-2.png)](./docs/img/online-boutique-frontend-2.png) |

## Service Architecture

**Online Boutique** is composed of many microservices written in different
languages that talk to each other over gRPC.

[![Architecture of
microservices](./docs/img/architecture-diagram.png)](./docs/img/architecture-diagram.png)

Find **Protocol Buffers Descriptions** at the [`./pb` directory](./pb).

| Service                                              | Language      | Description                                                                                                                       |
| ---------------------------------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| [frontend](./src/frontend)                           | Go            | Exposes an HTTP server to serve the website. Does not require signup/login and generates session IDs for all users automatically. |
| [cartservice](./src/cartservice)                     | C#            | Stores the items in the user's shopping cart in Redis and retrieves it.                                                           |
| [productcatalogservice](./src/productcatalogservice) | Go            | Provides the list of products from a JSON file and ability to search products and get individual products.                        |
| [currencyservice](./src/currencyservice)             | Node.js       | Converts one money amount to another currency. Uses real values fetched from European Central Bank. It's the highest QPS service. |
| [paymentservice](./src/paymentservice)               | Node.js       | Charges the given credit card info (mock) with the given amount and returns a transaction ID.                                     |
| [shippingservice](./src/shippingservice)             | Go            | Gives shipping cost estimates based on the shopping cart. Ships items to the given address (mock)                                 |
| [emailservice](./src/emailservice)                   | Python        | Sends users an order confirmation email (mock).                                                                                   |
| [checkoutservice](./src/checkoutservice)             | Go            | Retrieves user cart, prepares order and orchestrates the payment, shipping and the email notification.                            |
| [recommendationservice](./src/recommendationservice) | Python        | Recommends other products based on what's given in the cart.                                                                      |
| [adservice](./src/adservice)                         | Java          | Provides text ads based on given context words.                                                                                   |
| [loadgenerator](./src/loadgenerator)                 | Python/Locust | Continuously sends requests imitating realistic user shopping flows to the frontend.                                              |

## Features

- **[Kubernetes](https://kubernetes.io)/[GKE](https://cloud.google.com/kubernetes-engine/):**
  The app is designed to run on Kubernetes (both locally on "Docker for
  Desktop", as well as on the cloud with GKE).
- **[gRPC](https://grpc.io):** Microservices use a high volume of gRPC calls to
  communicate to each other.
- **[Istio](https://istio.io):** Application works on Istio service mesh.
- **[Ambassador Edge Stack](https://www.getambassador.io/):** Edge Management Technology
- **[OpenCensus](https://opencensus.io/) Tracing:** Most services are
  instrumented using OpenCensus trace interceptors for gRPC/HTTP.
- **[Skaffold](https://skaffold.dev):** Application
  is deployed to Kubernetes with a single command using Skaffold.
- **Synthetic Load Generation:** The application demo comes with a background
  job that creates realistic usage patterns on the website using
  [Locust](https://locust.io/) load generator.

## Installation

We offer the following deloyment methods:

1. **Running locally** (~20 minutes) You will build
   and deploy microservices images to a single-node Kubernetes cluster running
   on your development machine. There are three options to run a Kubernetes
   cluster locally for this demo:
   - [Minikube](https://github.com/kubernetes/minikube). Recommended for
     Linux hosts (also supports Mac/Windows).
   - [Docker for Desktop](https://www.docker.com/products/docker-desktop).
     Recommended for Mac/Windows.
   - [Kind](https://kind.sigs.k8s.io). Supports Mac/Windows/Linux.

1. **Running on Google Kubernetes Engine (GKE)”** (~30 minutes) You will build,
   upload and deploy the container images to a Kubernetes cluster on Google
   Cloud.


### Prerequisites

   - kubectl (can be installed via `gcloud components install kubectl`)
   - Local Kubernetes cluster deployment tool:
        - [Minikube (recommended for
         Linux)](https://kubernetes.io/docs/setup/minikube/)
        - [Docker for Desktop (recommended for Mac/Windows)](https://www.docker.com/products/docker-desktop)
          - It provides Kubernetes support as [noted
     here](https://docs.docker.com/docker-for-mac/kubernetes/)
        - [Kind](https://github.com/kubernetes-sigs/kind)
   - [skaffold]( https://skaffold.dev/docs/install/) ([ensure version ≥v1.10](https://github.com/GoogleContainerTools/skaffold/releases))

   - Istioctl: Istio configuration command line utility for service operators to debug and diagnose their Istio mesh.
     [Check the installation steps here](https://istio.io/latest/docs/setup/install/istioctl/)
   - Edgectl: Edge Control is the command-line tool for installing and managing the Ambassador Edge Stack
     [Check the installation steps here](https://www.getambassador.io/docs/latest/topics/using/edgectl/edge-control/)
   - (Optional): Kubectx and Kubens helps you switch between clusters and between namesapces (https://github.com/ahmetb/kubectx)

### Option 1: Running locally

> 💡 Recommended if you're planning to develop the application or giving it a
> try on your local cluster.

1. Launch a local Kubernetes cluster with one of the following tools:

    - To launch **Minikube** (tested with Ubuntu Linux). Please, ensure that the
       local Kubernetes cluster has at least:
        - 4 CPU's
        - 4.0 GiB memory
        - 32 GB disk space

      ```shell
      minikube start --cpus=4 --memory 4096 --disk-size 32g
      ```

    - To launch **Docker for Desktop** (tested with Mac/Windows). Go to Preferences:
        - choose “Enable Kubernetes”,
        - set CPUs to at least 3, and Memory to at least 6.0 GiB
        - on the "Disk" tab, set at least 32 GB disk space

    - To launch a **Kind** cluster:

      ```shell
      kind create cluster
      ```

2. Run `kubectl get nodes` to verify you're connected to “Kubernetes on Docker”.

3. Run `skaffold run` (first time will be slow, it can take ~20 minutes).
   This will build and deploy the application. If you need to rebuild the images
   automatically as you refactor the code, run `skaffold dev` command.

4. Run `kubectl get pods` to verify the Pods are ready and running.

5. Access the web frontend through your browser
    - **Minikube** requires you to run a command to access the frontend service:

    ```shell
    minikube service frontend-external
    ```

    - **Docker For Desktop** should automatically provide the frontend at http://localhost:80

    - **Kind** does not provision an IP address for the service.
      You must run a port-forwarding process to access the frontend at http://localhost:8080:

    ```shell
    kubectl port-forward deployment/frontend 8080:8080
    ```

### Option 2: Running on Google Kubernetes Engine (GKE)

> 💡 Recommended if you're using Google Cloud Platform and want to try it on
> a realistic cluster.

1.  Create a Google Kubernetes Engine cluster and make sure `kubectl` is pointing
    to the cluster.

    ```sh
    gcloud services enable container.googleapis.com
    ```

    ```sh
    gcloud container clusters create demo --enable-autoupgrade \
        --enable-autoscaling --min-nodes=3 --max-nodes=10 --num-nodes=5 --zone=us-central1-a
    ```

    ```
    kubectl get nodes
    ```

2.  Enable Google Container Registry (GCR) on your GCP project and configure the
    `docker` CLI to authenticate to GCR:

    ```sh
    gcloud services enable containerregistry.googleapis.com
    ```

    ```sh
    gcloud auth configure-docker -q
    ```

3.  In the root of this repository, run `skaffold run --default-repo=gcr.io/[PROJECT_ID]`,
    where [PROJECT_ID] is your GCP project ID.

    This command:

    - builds the container images
    - pushes them to GCR
    - applies the `./kubernetes-manifests` deploying the application to
      Kubernetes.

    **Troubleshooting:** If you get "No space left on device" error on Google
    Cloud Shell, you can build the images on Google Cloud Build: [Enable the
    Cloud Build
    API](https://console.cloud.google.com/flows/enableapi?apiid=cloudbuild.googleapis.com),
    then run `skaffold run -p gcb --default-repo=gcr.io/[PROJECT_ID]` instead.

4.  Find the IP address of your application, then visit the application on your
    browser to confirm installation.

        kubectl get service frontend-external

    **Troubleshooting:** A Kubernetes bug (will be fixed in 1.12) combined with
    a Skaffold [bug](https://github.com/GoogleContainerTools/skaffold/issues/887)
    causes load balancer to not to work even after getting an IP address. If you
    are seeing this, run `kubectl get service frontend-external -o=yaml | kubectl apply -f-`
    to trigger load balancer reconfiguration.


## Deploying and implementing Istio

> **Note:** if you followed GKE deployment steps above, run `skaffold delete` first to delete what's deployed.

1. Create a GKE cluster (described in "Option 2").

2. Install `istioctl` as described in the requirements section

3. Init `Istio` with a demo profil: `istioctl operator init `

4. Create the istio-system namespace: `kubectl create ns istio-system `

5. Install the automatic sidecar injection (annotate the `default` namespace
   with the label):

   ```sh
   kubectl label namespace default istio-injection=enabled
   ```

6. Apply the whitelist manifests in [`./istio-manifests`](./istio-manifests) directory.
   (This is required only once.)

   ```sh
   kubectl apply -f istio-manifests/whitelist-egress-googleapis.yaml
   ```

7. In the root of this repository, run `skaffold run --default-repo=gcr.io/[PROJECT_ID]`,
    where [PROJECT_ID] is your GCP project ID.

    This command:

    - builds the container images
    - pushes them to GCR
    - applies the `./kubernetes-manifests` deploying the application to
      Kubernetes.

8.  Run `kubectl get pods` to see pods are in a healthy and ready state.

9.  Explore the Observability feature of Istio using the `Kiali Dahsboard`:
    ```sh
    istioctl dashboard kiali
    ```
10. Explore Jaeger Dashboard to get an overview of the traffic tracing in the application:
    ```sh
    istioctl dashboard jaeger
    ```

## Deploying and implementing Ambassador Edge Stack

1. Install `edgectl` as described in the requirements section
2. Deploy Ambassador:
```sh
kubectl apply -f https://www.getambassador.io/yaml/aes-crds.yaml && \
kubectl wait --for condition=established --timeout=90s crd -lproduct=aes && \
kubectl apply -f https://www.getambassador.io/yaml/aes.yaml && \
kubectl -n ambassador wait --for condition=available --timeout=90s deploy -lproduct=aes
```
3. Deploy the deploying the quote service
   ```sh
   kubectl apply -f quote/
   ```
4. Get the Ambassador service endpoint:
   ```sh
   kubens ambassador
   kubectl get scv
   ```
   Copy the `ambassador` LoadBalancer IP

5. Explore the Ambassador features using kubectl and the Ambassador Admin Dashboard:
   ```
   edgectl login --namespace=ambassador <ambassador_ip>
   ```
6. Explore Developer Onboarding feature

The Quote service we just deployed publishes its API as a Swagger document. This API is automatically detected by the Ambassador Edge Stack and published.

In the Edge Policy Console, navigate to the APIs tab. You'll see the documentation there for internal use.

Navigate to https://<IP address>/docs/ to see the externally visible Developer Portal (make sure you include the trailing /). This is a fully customizable portal that you can share with third parties who need information about your APIs.

## Cleanup

If you've deployed the application with `skaffold run` command, you can run
`skaffold delete` to clean up the deployed resources.

If you've deployed the application with `kubectl apply -f [...]`, you can
run `kubectl delete -f [...]` with the same argument to clean up the deployed
resources.

## Conferences featuring Online Boutique

- [Google Cloud Next'18 London – Keynote](https://youtu.be/nIq2pkNcfEI?t=3071)
  showing Stackdriver Incident Response Management
- Google Cloud Next'18 SF
  - [Day 1 Keynote](https://youtu.be/vJ9OaAqfxo4?t=2416) showing GKE On-Prem
  - [Day 3 – Keynote](https://youtu.be/JQPOPV_VH5w?t=815) showing Stackdriver
    APM (Tracing, Code Search, Profiler, Google Cloud Build)
  - [Introduction to Service Management with Istio](https://www.youtube.com/watch?v=wCJrdKdD6UM&feature=youtu.be&t=586)
- [KubeCon EU 2019 - Reinventing Networking: A Deep Dive into Istio's Multicluster Gateways - Steve Dake, Independent](https://youtu.be/-t2BfT59zJA?t=982)

---

