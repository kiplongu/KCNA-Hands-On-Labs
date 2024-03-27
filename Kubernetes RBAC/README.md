# Important commands

kubectl create clusterrole cluster-superhero --verb="*" --resource="*"

kubectl create rolebinding cluster-superhero --clusterrole=cluster-superhero --group=cluster-superheroes


verify

kubectl auth can-i '*' '*'


![K8S RBAC](image.png)

