Kubernetes Storage - Study Tips
For the KCNA Examination, as well as having top level knowledge there is a heavy emphasis on the internals and specifics of the Kubernetes Storage subsystem. Please pay attention to the following areas -

What is Ephemeral Storage

What is Persistent Storage

Examples of CNCF Graduated Storage Solution(s)

Retain Policies

# Ephemeral Storage

We'll begin by capturing ubuntu pod yaml with sleep infinity as a command -

kubectl run --image=ubuntu ubuntu -o yaml --dry-run=client --command sleep infinity | tee ubuntu_emptydir.yaml

Update this specification so that it includes a volumeMount and an emptyDir volume -

cat <<EOF > ubuntu_emptydir.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ubuntu
  name: ubuntu
spec:
  containers:
  - command:
    - sleep
    - infinity
    image: ubuntu
    name: ubuntu
    resources: {}
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: cache-volume
    emptyDir: {
      medium: Memory,
    }
status: {}
EOF

And let's apply this file -

kubectl apply -f ubuntu_emptydir.yaml

And lets check our pod is running -

kubectl get pods -o wide

We'll hop into this pod with an interactive shell -

kubectl exec -it ubuntu -- bash

And if we go into cache and do a df -h, we can see that the filesytem being used is tmpfs -

cd cache; df -h .

With this being a memory based filesystem, it will be highly performant. Let's test this using dd -

dd if=/dev/zero of=output oflag=sync bs=1024k count=1000

For now, let's exit the pod -

exit

When we get rid of this container it will also remove the emptyDir volume -

kubectl delete pod/ubuntu --now


# Persistent Storage
Let's take a look at the storageclasses available in our cluster, we will have a single storageclass and it will be marked as default -

kubectl get storageclass

Familiarise yourself with the storageclass specication, in particular the reclaimPolicy section -

kubectl explain storageclass | more

We'll create a persistent volume using the k3s storageClass of local-path -

cat <<EOF > manual_pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: manual-pv001
spec:
  storageClassName: local-path
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/var/lib/rancher/k3s/storage/manual-pv001"
    type: DirectoryOrCreate
EOF

And we will apply this file -

kubectl apply -f manual_pv.yaml

Check the available persistent volumes and note that the reclaim policy is set to Retain -

kubectl get pv

Let's now create a manual persistent volume claim against this persistent volume -

cat <<EOF > manual_pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: manual-claim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Gi
  storageClassName: local-path
  volumeName: manual-pv001
EOF

Apply this file -

kubectl apply -f manual_pvc.yaml

And check the persistent volume claim -

kubectl get persistentvolumeclaim

This can also be done the short way -

kubectl get pvc



# Dynamic PVC
Let's create a dynamic version of our Persistent Volume Claim, this will be similar to our manual approach but without a volumeName entry -

cat <<EOF > dynamic_pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-claim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Gi
  storageClassName: local-path
EOF

Let's apply this -

kubectl apply -f dynamic_pvc.yaml

And if we get our pvc's you'll see that this is Pending -

kubectl get pvc

If we run a describe on the pvc, we will see that it is awaiting a consumer before it is bound -

kubectl describe pvc/dynamic-claim

And if we check our pv's, we only have the manual pv -

kubectl get pv

Let's mock up a ubuntu pod with sleep infinity as a starting base -

kubectl run --image=ubuntu ubuntu -o yaml --dry-run=client --command sleep infinity | tee ubuntu_with_volumes.yaml

And we'll modify this to include volumeMounts and volumes for both our manual and dynamic pvc's -

cat <<EOF > ubuntu_with_volumes.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ubuntu
  name: ubuntu
spec:
  containers:
  - command:
    - sleep
    - infinity
    image: ubuntu
    name: ubuntu
    resources: {}
    volumeMounts:
    - mountPath: /manual
      name: manual-volume
    - mountPath: /dynamic
      name: dynamic-volume
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
    - name: manual-volume
      persistentVolumeClaim:
        claimName: manual-claim
    - name: dynamic-volume
      persistentVolumeClaim:
        claimName: dynamic-claim
status: {}
EOF

As the k3s storageclass provisioner is very basic, we should use a nodeSelector to guide this pod to a specific node, check the labels for node/worker-1 -

kubectl describe node/worker-1 | more

And update the yaml to include a nodeSelector field -

cat <<EOF > ubuntu_with_volumes.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ubuntu
  name: ubuntu
spec:
  nodeSelector:
    kubernetes.io/hostname: worker-1
  containers:
  - command:
    - sleep
    - infinity
    image: ubuntu
    name: ubuntu
    resources: {}
    volumeMounts:
    - mountPath: /manual
      name: manual-volume
    - mountPath: /dynamic
      name: dynamic-volume
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
    - name: manual-volume
      persistentVolumeClaim:
        claimName: manual-claim
    - name: dynamic-volume
      persistentVolumeClaim:
        claimName: dynamic-claim
status: {}
EOF

And we can apply this file -

kubectl apply -f ubuntu_with_volumes.yaml

Check that the pod is running -

kubectl get pod

Check pv's, A dynamic pv will have been created, notice the reclaim policies for both -

kubectl get pv

Check pvc's, they should now both be bound -

kubectl get pvc

And let's write data to our volumes, access the pod with an interactive shell -

kubectl exec -it ubuntu -- bash

And if we check both of these, they are mounted volumes -

cd /manual; df -h .; cd /dynamic; df -h .

Let's write a text file to each of these and we'll exit -

echo hello > /manual/test.txt; echo hello > /dynamic/test.txt; exit

As these are persistent volumes, if we delete the pod and recreate it -

kubectl delete pod/ubuntu --now && kubectl apply -f ubuntu_with_volumes.yaml

And again access this pod -

kubectl exec -it ubuntu -- bash

We can see that our files will still exist as our volumes are persistent -

cat /manual/test.txt; cat /dynamic/test.txt

Let's exit -

exit

Before we go, let's see what happens when we delete the pod and pvc's -

kubectl delete pod/ubuntu pvc/dynamic-claim pvc/manual-claim --now

If we check our pv's, the manual one still exists as the Reclaim policy is set to Retain -

kubectl get pv

Finally we'll clean this up and remove our files -

kubectl delete pv/manual-pv001 --now; rm -rf dynamic_pvc.yaml manual_pv* ubuntu_emptydir.yaml ubuntu_with_volumes.yaml


