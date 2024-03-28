# Kubernetes Scheduling and NodeName - Study Tips
For the KCNA qualification, a basic knowledge and understanding of the role of the scheduler is required. Particular attention should be applied to the use of nodeName as well as nodeSelector and the mechanism of labels that the nodeSelector makes us of (nodeSelector is an addition covered in the Further Study addendum).

# SchedulerName
Let's begin by templating an nginx pod that we'll later target with a custom scheduler -

kubectl run nginx --image=nginx -o yaml --dry-run=client | tee nginx_scheduler.yaml

We are interested in a pod.spec option known as schedulerName, use kubectl explain to see more information on this -

kubectl explain pod.spec | more

Update the nginx_scheduler.yaml file to include a schedulerName for a scheduler called my-scheduler -

cat <<EOF > nginx_scheduler.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  schedulerName: my-scheduler
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
EOF

And let's apply this file -

kubectl apply -f nginx_scheduler.yaml

Our pod will be stuck in pending as there is no scheduler called my-scheduler running -

kubectl get pods -o wide

We're going to make use of a convenient code example that mimicks the functionality of a scheduler as a bash script, firstly install git and jq if required -

apt update && apt install -y git jq

And lets clone this project -

git clone https://github.com/spurin/simple-kubernetes-scheduler-example.git

And lets change to the directory and take a look at this file -

cd simple-kubernetes-scheduler-example; more my-scheduler.sh

And if you're interested in seeing how the json traversal in the script works, take a look at the json as follows -

kubectl get nodes -o json

If we run this scheduler, it should schedule our pending pod, press ctrl-c when done -

./my-scheduler.sh

And if we check our pods, this will now be scheduled -

kubectl get pods -o wide

For now, let's clean this up -

kubectl delete pod/nginx --now