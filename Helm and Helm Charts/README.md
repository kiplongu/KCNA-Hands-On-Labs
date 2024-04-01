# Helm and Helm Charts - Study Tips
For the KCNA exam a basic understanding of Helm as a package manager solution for Kubernetes is required.


Before we begin, ensure you have git installed, as it's required for Helm's plugin installation. We'll start by installing git, and at the same time we also going to install tree for a clearer view of directory structures used in Helm -

apt update && apt install -y git tree

Helm installation can be done in various ways. For convenience in this lab, we will use the get_helm.sh script. Normally, you should inspect scripts before executing them, but for this tutorial, we'll safely pipe the script to bash directly. Let's install Helm now -

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

Verify that Helm is installed and available in your path by checking its version. This command will show the installed Helm version details -

helm version

Let's create a Helm chart for the Flappy Bird style application. This process generates a basic structure for our application. Run the following command to create a new Helm chart named "flappy-app" -

helm create flappy-app

Navigate into the newly created flappy-app directory to explore the Helm chart's structure. This will help us understand the components of a Helm chart -

cd flappy-app

Let's examine the directory structure of our Helm chart using the tree command. This gives us an overview of the files and directories in our chart -

tree

Now, customize the Chart.yaml file. This file contains key information about our Helm chart, such as the chart name and description. Edit the file to set the description to Flappy Dock Game Helm chart (without quotes) and leave the version as it's default -

vim Chart.yaml

Modify the values.yaml file to set the image repository to spurin/flappy-dock and use the latest tag (again without quotes for both). Also, disable the serviceAccount by setting its related boolean values to false -

vim values.yaml

Package the Helm chart for distribution. This step demonstrates Helm's versatility in chart management and version control -

helm package .

Deploy the application using the packaged Helm chart. This showcases Helm's ability to manage applications from packaged charts, which is useful for version control and distribution -

helm install flappy-app ./flappy-app-0.1.0.tgz

Helm would have provided us with some convenient commands, for ease we can run these as follows, this will also set up a port-forward to our application (you may need to retry if the app isn't running) -

export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=flappy-app,app.kubernetes.io/instance=flappy-app" -o jsonpath="{.items[0].metadata.name}"); export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}"); echo "Visit http://127.0.0.1:8080 to use your application"; kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT

Bring up a new tab to the Reverse Proxy and select the following - Control Plane ➜ 127.0.0.1 ➜ 8080 and then press Go, have some fun with the game and when complete, move back to the previous tab and press ctrl-c

Explore the deployed Kubernetes resources to understand how Helm interacts with Kubernetes. Check the deployment, pods, and services created by our Helm chart -

kubectl get deployment; echo; kubectl get pods; echo; kubectl get svc

Check whats showing from a helm viewpoint -

helm list

And then, let's uninstall the Helm chart to clean up the deployed resources -

helm uninstall flappy-app

Finally, let's clean up the environment -

cd ..; rm -rf flappy-app

Congratulations on completing the Helm lab! You have learned how to install Helm, create and customize a Helm chart, package and deploy an application, and clean up resources. These skills are valuable for managing Kubernetes workloads efficiently. The next two videos on Service Meshes and Observability are theory only, after that the next lab will be Prometheus and Grafana 🚀