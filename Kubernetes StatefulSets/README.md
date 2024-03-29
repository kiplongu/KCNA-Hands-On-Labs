# Kubernetes StatefulSets

There isn't a quick way of creating a StatefulSet from the CLI but a Deployment gets us very close, we'll mock up a Deployment and tee to a file -

kubectl create deployment nginx --image=nginx --replicas=3 --dry-run=client -o yaml | tee statefulset.yaml

And we'll modify this, to change it to a StatefulSet, we're also going to include a reference to serviceName -

cat <<EOF > statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  serviceName: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
EOF

And let's apply this file -

kubectl apply -f statefulset.yaml

And lets check our statefulset -

kubectl get statefulset

If we check our pods, these will have been named accordingly -

kubectl get pods -o wide

*Bonus Lab Content* - If a pod in a statefulset was removed, causing a restart, whilst the ip may change, the name will stay consistent. Delete pod/nginx-2 -

kubectl delete pod/nginx-2 --now

And now query our pods -

kubectl get pods -o wide

Let's make use of the statefulsets service functionality, we'll create a headless service called nginx -

kubectl create service clusterip --clusterip=None nginx

Check the service is visible -

kubectl get service

And if we take a look at, we can see our endpoint slices -

kubectl get endpoints

The yaml output is great for viewing endpoints -

kubectl get endpoints/nginx -o yaml

Let's check this from the viewpoint of a pod, we'll run a curlimages/curl container -

kubectl run --rm -i --tty curl --image=curlimages/curl:8.4.0 --restart=Never -- sh

And let's curl our statefulset service name -

curl nginx-1.nginx.default.svc.cluster.local

And we'll exit this container -

exit

Take a look at the statefulset yaml, you'll notice that the updateStrategy/rollingUpdate has a partition value of 0 -

kubectl get statefulset/nginx -o yaml | more

In the accompanying video we edit our existing statefulset to change the partition value to 2 but, the following command will do the equivalent -

kubectl patch statefulset/nginx -p '{"spec":{"updateStrategy":{"rollingUpdate":{"partition":2}}}}'

We will set the image to nginx:alpine and watch the rollout -

kubectl set image statefulset/nginx nginx=nginx:alpine && kubectl rollout status statefulset/nginx

Because of the partition value, the rollout only took place on nginx-2, we can verify this with the following commands (check the image) -

kubectl describe pod/nginx-1 | more

kubectl describe pod/nginx-2 | more


# StatefulSet Persistent Storage
Update our statefulset to include a dynamic volume -

cat <<EOF > statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: nginx
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: nginx
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "local-path"
      resources:
        requests:
          storage: 1Gi
EOF

And we will recycle our statefulset -

kubectl delete -f statefulset.yaml && kubectl apply -f statefulset.yaml

Watch our pods start, press ctrl-c to exit -

watch --differences kubectl get pods -o wide

And we can verify that we have an individual pvc and pv for each pod -

kubectl get pvc

kubectl get pv

We'll execute a shell into nginx-0 -

kubectl exec -it nginx-0 -- bash

Create some data -

cd /data; touch this_is_nginx-0; ls -l

And exit -

exit

Let's delete the pod and watch it be recreated -

kubectl delete pod/nginx-0 --now; watch --differences kubectl get pods -o wide

And if we execute a shell again -

kubectl exec -it nginx-0 -- bash

We can verify that our data still exists -

ls /data

Let's exit -

exit

And we can take this further, we can delete the entire statefulset -

kubectl delete -f statefulset.yaml

Our pvc and pv will still exist -

kubectl get pvc; kubectl get pv

And if we apply the statefulset again -

kubectl apply -f statefulset.yaml

Check our pods are running -

kubectl get pods -o wide

And exec back into the pod -

kubectl exec -it nginx-0 -- bash

And check our data, we can confirm that it is available as expected -

ls /data

Let's exit -

exit

And we'll cleanup -

kubectl delete statefulset/nginx --now; for i in 0 1 2; do kubectl delete pvc/nginx-nginx-$i --now; done; kubectl delete service/nginx; rm statefulset.yaml

