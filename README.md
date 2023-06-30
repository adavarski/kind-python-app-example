## Simple python:flask app (for playgrounds & LABS)  with Docker & Kubernetes

This is a simple Flask application with a Dockerfile and Kubernetes manifests to help you deploy it locally in 3 different ways.
- Python & Flask on localhost.
- Docker container.
- Kubernetes in kind cluster.

## Requirements

- Python 3 installed (if you want to run the application locally)
- Docker installed
- [Kind installed](https://kind.sigs.k8s.io/)
- [Kind ingress configured](https://kind.sigs.k8s.io/docs/user/ingress/)

## Installation

1. Clone the repository to your local machine.
2. Open a terminal and navigate to the root directory of the project.
3. If you want to run the application locally, install the Flask dependencies by running the following command:

```
pip3 install -r requirements.txt
```

4. To run the application locally, execute the following command from the project root directory:

```
python3 app/main.py
```

5. To run the application using Docker, you need to first build the Docker image by running the following command from
   the project root directory:

```
docker build -t my-flask-app .
```

6. After building the Docker image, you need to tag it by running the following command:

```
### tag image 
docker tag my-flask-app:latest davarski/my-flask-app:latest

### push the image to your personal docker registry
docker push davarski/my-flask-app:latest
```   

Note: Replace `davarski` with your Docker registry name.

7. To run the application using Docker, execute the following command:

```
docker run -p 5001:5000 davarski/my-flask-app
```

Note: Replace `davarski` with your Docker registry name.

8. If you want to run the application in a Kubernetes cluster using Kind, first create a Kind cluster using the ingress
   configuration at https://kind.sigs.k8s.io/docs/user/ingress/

```
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF

```

9. (OPTIONAL) After creating the Kind cluster, you need to load the Docker image into the Kind cluster by running the following

```
kind load docker-image davarski/my-flask-app:latest --name kind
```

Note: Replace `davarski` with your Docker registry name.


10. Ingress NGINX
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

11. To deploy the application to the Kind cluster, apply the deployment.yaml file by running the following command:

```
kubectl apply -f manifest.yaml
```

The manifest contain the deployment configuration, service and ingress. Verify you have all the resources by running:

```
kubectl get all
kubectl get ingress

```

You should be able to access the application at your `localhost`:

```
$ kubectl get ingress
NAME            CLASS    HOSTS   ADDRESS     PORTS   AGE
flask-ingress   <none>   *       localhost   80      81s
$ curl localhost
Hello from Python!

```
Clean environment:
```
kind delete cluster
```
