# Important commands

kubectl create clusterrole cluster-superhero --verb="*" --resource="*"

kubectl create rolebinding cluster-superhero --clusterrole=cluster-superhero --group=cluster-superheroes


verify

kubectl auth can-i '*' '*'


![K8S RBAC](image.png)

# Configuring an RBAC User/Group Manually

We'll see how we can configure a user/group for use with RBAC from scratch, first we'll start by generating an rsa private key for our user batman -

openssl genrsa -out batman.key 4096

We'll now create a certificate signing request, we'll reference our private key, an output file and we'll specify the subject information with CN and O for our user and group -

openssl req -new -key batman.key -out batman.csr -subj "/CN=batman/O=cluster-superheroes" -sha256

And if you wish you can review the files -

cat batman.key

cat batman.csr

We'll now structure a CSR signing request, first we'll create variables with our data -

CSR_DATA=$(base64 batman.csr | tr -d '\n')

CSR_USER=batman

And if we check these, both will now be set -

echo $CSR_DATA

echo $CSR_USER

Let's template a Kubernetes Certificate Signing Request as yaml, we'll use placeholders for our variables and will redirect to a file -

cat <<EOF > batman-csr-request.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: ${CSR_USER}
spec:
  request: ${CSR_DATA}
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF

And if we check our file, we will see these entries populated -

cat batman-csr-request.yaml

We'll apply this request -

kubectl apply -f batman-csr-request.yaml

And if we check in Kubernetes we will see this certificate signing request as pending -

kubectl get certificatesigningrequest

We can use the shorthand of csr which is more commonly used in these circumstances -

kubectl get csr

As our current super user we'll approve this request -

kubectl certificate approve batman

And if we check again, we will have an approved csr -

kubectl get csr

If we query the signed certificate using json as output, we will be able to see our certificate in base64 -

kubectl get csr batman -o json

To directly capture this entry, we'll use jsonpath and will tell it to walk the path of status, then certificate -

kubectl get csr batman -o jsonpath='{.status.certificate}'

At the moment this is base64 so let's decode this -

kubectl get csr batman -o jsonpath='{.status.certificate}' | base64 -d

And we'll redirect this to a file -

kubectl get csr batman -o jsonpath='{.status.certificate}' | base64 -d > batman.crt

And now we have the following files -

ls -l

We can use openssl to decode the approved/signed certificate, showing that the subject line contains CN and O -

openssl x509 -in batman.crt -text -noout

Adding Information to a Kubeconfig file
Let's use our existing kubeconfig file as a template -

cp /root/.kube/config batman-clustersuperheroes.config

If we take a look, this currently has a lot of information that we don't want our batman user to have -

cat batman-clustersuperheroes.config

Let's clean this file, we'll use the KUBECONFIG variable and we'll unset users.default -

KUBECONFIG=batman-clustersuperheroes.config kubectl config unset users.default

We'll delete the current context -

KUBECONFIG=batman-clustersuperheroes.config kubectl config delete-context default

And we'll unset the current context -

KUBECONFIG=batman-clustersuperheroes.config kubectl config unset current-context

Our base file is now simplified -

cat batman-clustersuperheroes.config

Let's embed the new information, first set-credentials as batman, we'll pass the certificate data and the private key, we'll also embed this information -

KUBECONFIG=batman-clustersuperheroes.config kubectl config set-credentials batman --client-certificate=batman.crt --client-key=batman.key --embed-certs=true

Set the default context to use the default cluster with a user of batman -

KUBECONFIG=batman-clustersuperheroes.config kubectl config set-context default --cluster=default --user=batman

And use the context of default -

KUBECONFIG=batman-clustersuperheroes.config kubectl config use-context default

Now our configuration files looks like the following -

cat batman-clustersuperheroes.config

And we can test that this is working, we'll be able to get nodes, create a pod and delete a pod, essentially our user is a super user -

KUBECONFIG=batman-clustersuperheroes.config kubectl get nodes

KUBECONFIG=./batman-clustersuperheroes.config kubectl run nginx --image=nginx

KUBECONFIG=./batman-clustersuperheroes.config kubectl delete pod/nginx --now

