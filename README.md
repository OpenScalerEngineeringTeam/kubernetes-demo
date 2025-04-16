### Online Boutique Kubernetes Demo

Online Boutique is a Kubernetes demo application derived from the [GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo) repository, full credit to its contributors.

If you like this demo, please ★Star [the original repository](https://github.com/GoogleCloudPlatform/microservices-demo) to show your interest!

### Description

<!-- <p align="center">
<img src="/src/frontend/static/icons/Hipster_HeroLogoMaroon.svg" width="300" alt="Online Boutique" />
</p> -->

**Online Boutique** is a cloud-first microservices demo application. The application is a
web-based e-commerce app where users can browse items, add them to the cart, and purchase them.

## Architecture

**Online Boutique** is composed of 11 microservices written in different
languages that talk to each other over gRPC.

[![Architecture of
microservices](/docs/img/architecture-diagram.png)](/docs/img/architecture-diagram.png)

Find **Protocol Buffers Descriptions** at the [`./protos` directory](/protos).

| Service                                             | Language      | Description                                                                                                                       |
| --------------------------------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| [frontend](/src/frontend)                           | Go            | Exposes an HTTP server to serve the website. Does not require signup/login and generates session IDs for all users automatically. |
| [cartservice](/src/cartservice)                     | C#            | Stores the items in the user's shopping cart in Redis and retrieves it.                                                           |
| [productcatalogservice](/src/productcatalogservice) | Go            | Provides the list of products from a JSON file and ability to search products and get individual products.                        |
| [currencyservice](/src/currencyservice)             | Node.js       | Converts one money amount to another currency. Uses real values fetched from European Central Bank. It's the highest QPS service. |
| [paymentservice](/src/paymentservice)               | Node.js       | Charges the given credit card info (mock) with the given amount and returns a transaction ID.                                     |
| [shippingservice](/src/shippingservice)             | Go            | Gives shipping cost estimates based on the shopping cart. Ships items to the given address (mock)                                 |
| [emailservice](/src/emailservice)                   | Python        | Sends users an order confirmation email (mock).                                                                                   |
| [checkoutservice](/src/checkoutservice)             | Go            | Retrieves user cart, prepares order and orchestrates the payment, shipping and the email notification.                            |
| [recommendationservice](/src/recommendationservice) | Python        | Recommends other products based on what's given in the cart.                                                                      |
| [adservice](/src/adservice)                         | Java          | Provides text ads based on given context words.                                                                                   |
| [loadgenerator](/src/loadgenerator)                 | Python/Locust | Continuously sends requests imitating realistic user shopping flows to the frontend.                                              |

## Screenshots

| Home Page                                                                                                             | Checkout Screen                                                                                                        |
| --------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| [![Screenshot of store homepage](/docs/img/online-boutique-frontend-1.png)](/docs/img/online-boutique-frontend-1.png) | [![Screenshot of checkout screen](/docs/img/online-boutique-frontend-2.png)](/docs/img/online-boutique-frontend-2.png) |

## Quickstart

1. Ensure you have the following requirements:
   - Shell environment with `git`, and `kubectl`.
   - An [OpenScaler Kubernetes Cluster](https://platform.alpha.openscaler.net/kubernetes) with at least 8GB of RAM in total for _worker_ nodes (see [How to create an OpenScaler Kubernetes cluster](https://openscaler.net/docs/services/kubernetes))
   - ⚠️ Important: by default, only worker nodes run your services.If you're using a single node or two nodes with 4GB RAM, make sure to enable task scheduling on all nodes. Learn how [here](https://openscaler.net/docs/services/kubernetes/access-k8s-cluster/terminal#allow-scheduling-on-single-node-cluster).
2. Clone the latest major version.

   ```sh
   git clone --depth 1 --branch openscaler https://github.com/OpenScalerEngineeringTeam/kubernetes-demo.git
   cd kubernetes-demo/
   ```

   The `--depth 1` argument skips downloading git history.

3. Make sure you have your Kubernetes cluster credentials set (see how to [get access to your cluster](https://openscaler.net/docs/services/kubernetes/access-k8s-cluster)).

4. Deploy Online Boutique to the cluster :

   From the root folder of this repository, navigate to the kustomize/ directory.

   ```bash
   cd kustomize/
   ```

   See what the default Kustomize configuration defined by kustomize/kustomization.yaml will generate (without actually deploying them yet).

   ```bash
   kubectl kustomize .
   ```

   Apply the default Kustomize configuration (kustomize/kustomization.yaml).

   ```bash
   kubectl apply -k .
   ```

5. Wait for the pods to be ready.

   ```sh
   kubectl get pods
   ```

   After a few minutes, you should see the Pods in a `Running` state:

   ```
   NAME                                     READY   STATUS    RESTARTS   AGE
   adservice-76bdd69666-ckc5j               1/1     Running   0          2m58s
   cartservice-66d497c6b7-dp5jr             1/1     Running   0          2m59s
   checkoutservice-666c784bd6-4jd22         1/1     Running   0          3m1s
   currencyservice-5d5d496984-4jmd7         1/1     Running   0          2m59s
   emailservice-667457d9d6-75jcq            1/1     Running   0          3m2s
   frontend-6b8d69b9fb-wjqdg                1/1     Running   0          3m1s
   loadgenerator-665b5cd444-gwqdq           1/1     Running   0          3m
   paymentservice-68596d6dd6-bf6bv          1/1     Running   0          3m
   productcatalogservice-557d474574-888kr   1/1     Running   0          3m
   recommendationservice-69c56b74d4-7z8r5   1/1     Running   0          3m1s
   redis-cart-5f59546cdd-5jnqf              1/1     Running   0          2m58s
   shippingservice-6ccc89f8fd-v686r         1/1     Running   0          2m58s
   ```

6. You now have two options to access your Online Boutique application :
   - Option 1 : Access Application with Ingress
   - Option 2 : Access Application locally with port forwarding

### Access Application locally with port forwarding

1. Forward the frontend service to your local machine

   ```bash
   kubectl port-forward svc/frontend 8080:80   # --address 0.0.0.0  flag to make accessible in all your local network
   ```

2. Done! You may now access your Online Boutique application at `http://localhost:8080` (or `http://<your-local-ip>:8080` if you used the `--address 0.0.0.0` flag)

### Access Application with Ingress

1. Head to your Kubernetes cluster page [choose your cluster from the list](https://platform.alpha.openscaler.net/kubernetes)
2. Create a LoadBalancer for your cluster from the "Network" tab and install your Ingress Controller (as described in the "Network" tab)
3. Find your load balancer's HTTP port (for example `9039`) (follow guide in the "Network" tab)
4. Done! you can now access your Online Boutique application at `http://k8s.alpha.openscaler.net:YOUR_INGRESS_PORT` (replace `YOUR_INGRESS_PORT` with the port you found in the previous step)
