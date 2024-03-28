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

