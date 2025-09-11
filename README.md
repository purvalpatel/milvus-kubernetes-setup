In standalone milvus it is lost connection when so many number of data is inserted in batch. I.e. around more then 1000+. so in that case we required distributed setup. <br>
Ref link - https://milvus.io/docs/install_cluster-milvusoperator.md

#### Helm installation
```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null

sudo apt-get install apt-transport-https --yes

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list 
sudo apt-get update

sudo apt-get install helm
```

#### Create Storageclass:
storageclass.yaml
```YAML
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-path
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
```

#### Local storage path plugin:
```
https://raw.githubusercontent.com/purvalpatel/milvus-kubernetes-setup/7b55267b22ba4c077b0e0d58604205a5fe28d978/values.yaml
```
Note: <br>
By default the storage class location is /opt/local-path-provisioner. If you wanted to change it to /Data2/local-path-provisioner. <br>

##### Edit custom the local-path-storage.
```
kubectl -n local-path-storage edit configmap local-path-config
```
Edit with : `/custom-storage/kubernetes-deployment/milvus/data` <br>

Now restart the pod, <br>
```
kubectl -n local-path-storage delete pod -l app=local-path-provisioner
```

Note: if Using dynamic storage class. <br>
You only create a PVC (the request).<br>
Kubernetes + the provisioner (like local-path-provisioner, nfs-subdir-external-provisioner, AWS EBS, Ceph, etc.) will auto-create a PV that matches your PVC. <br>
  
So you dont need to create PV manually.	<br>

PVC are automatically created according to values.yaml settings. <br>

#### Create values.yaml
refere `values.yaml`

#### Remove taint:
Remove taint from master node so, pods can be scheduled on control plane.
```
kubectl taint nodes <node-name> node-role.kubernetes.io/control-plane-
```

#### Create service:
```
helm install milvus milvus/milvus -n milvus-operator -f values.yaml
```
#### check all pods status:
```
kubectl get all -n milvus-operator
kubectl get pods -n milvus-operator
```

#### Setup milvus dashboard:
please refer `attu.yaml`

```
kubectl apply -f attu.yaml
```

#### setup minio service:
please refere `minio-service.yaml`

```
kubectl apply -f minio-service.yaml
```

#### Setup milvus service:
please refere `milvus-service.yaml`
```
kubectl apply -f milvus-service.yaml
```

#### Other commands:
Rollout new version of milvus with helm:

```
helm upgrade milvus milvus/milvus -n milvus-operator -f values.yaml
```
Setup is completed.
