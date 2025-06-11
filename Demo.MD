1. Prerequisites (Assumed installed)

- You already have:
   - **minikube** (host cluster) -> [cluster.MD](./cluster.MD)
   - **kubectl**
   - **vcluster CLI**
   - Optionally: install **kubectx** for easy context switching:

2. Create a virtual cluster
- Run the following command to create a virtual cluster inside your Minikube:
  - ```vcluster create tenant-a-vcluster --namespace tenant-a```
    - What this does:
       - Creates a Kubernetes namespace: tenant-a
       - Installs vCluster in that namespace
       - Adds kubeconfig context for the virtual cluster
       - Switches your current context to the virtual cluster

3. Verify virtual cluster context
- Check that you are now connected to the virtual cluster:
  - ```kubectl config current-context```
    - Output : ```vcluster_tenant-a-vcluster_tenant-a_host-k8s-cluster```
- Then check namespaces inside the virtual cluster:
```
kubectl get ns
NAME              STATUS   AGE
default           Active   6m53s
kube-node-lease   Active   6m53s
kube-public       Active   6m53s
kube-system       Active   6m53s
```


4. Check from the host (Minikube)
- Switch back to Minikube context:
  - ```kubectx host-k8s-cluster```
- Check namespaces:
  - ```kubectl get ns```
    - You’ll see a new namespace ```tenant-a```.

```
NAME              STATUS   AGE
default           Active   17m
ingress-nginx     Active   17m
kube-node-lease   Active   17m
kube-public       Active   17m
kube-system       Active   17m
tenant-a          Active   10m
```

- Check the pods inside it:
    - ```kubectl get pods -n tenant-a```
      - You’ll see:
        - vCluster pod (control plane + syncer)
        - coredns pod (synced from virtual cluster)
```
NAME                                                       READY   STATUS    RESTARTS   AGE
coredns-94f599b5-sj9kq-x-kube-system-x-tenant-a-vcluster   1/1     Running   0          11m
tenant-a-vcluster-0                                        1/1     Running   0          11m
```

5. Deploy a sample workload inside the virtual cluster
- Switch context to the vCluster again:
  - ```kubectx vcluster_tenant-a-vcluster_tenant-a_host-k8s-cluster```
- Create an NGINX pod:
  - ```kubectl run nginx --image=nginx```


6. View the synced pod in the host cluster
- Switch back to Minikube:
  - ```kubectx host-k8s-cluster```
- Check the pod name in tenant-a namespace:
  - ```kubectl get pods -n tenant-a```
  - You’ll see something like:
    - ```nginx-x-default-x-tenant-a-vcluster```
    - This is the same nginx pod, synced from virtual to host cluster.

```
NAME                                                       READY   STATUS    RESTARTS   AGE
coredns-94f599b5-sj9kq-x-kube-system-x-tenant-a-vcluster   1/1     Running   0          17m
nginx-x-default-x-tenant-a-vcluster                        1/1     Running   0          44s
tenant-a-vcluster-0                                        1/1     Running   0          18m
```

7. Test pod networking
- Still in host context:
- Get nginx pod IP:
  - ```NGINX_IP=$(kubectl get pod nginx-x-default-x-tenant-a-vcluster -n tenant-a -o jsonpath='{.status.podIP}')```
- Now test connectivity:
  - ```kubectl run temp-pod --rm -it --restart=Never --image=curlimages/curl -- curl $NGINX_IP```
  - Expected: You’ll see nginx HTML output like:

```
<!DOCTYPE html>
<html>
<head><title>Welcome to nginx!</title></head>
...
```

8. Test CRDs inside vCluster
- Switch back to vCluster context:
  - ```kubectx vcluster_tenant-a-vcluster_tenant-a_host-k8s-cluster```
- Apply a CRD inside vCluster:
  - ```kubectl apply -f https://raw.githubusercontent.com/Liquid-Reply/vcluster-idp-tools/refs/heads/main/crds/crontabs.yaml```

- Now apply a Custom Resource (CR):
  - ```kubectl apply -f https://raw.githubusercontent.com/Liquid-Reply/vcluster-idp-tools/refs/heads/main/resources/crontab.yaml```
  - Output:
   - ```crontab.stable.example.com/my-new-cron-object created```

9. Try using the same CR in the host (and fail)
- Switch to Minikube:
  - ```kubectx host-k8s-cluster```
- ```kubectl apply -f https://raw.githubusercontent.com/Liquid-Reply/vcluster-idp-tools/refs/heads/main/resources/crontab.yaml```
 - Expected:
   - ```error:  no matches for kind CronTab in version stable.example.com/v1 ensure CRDs are installed first```
   - Why? Because the CRD exists only in the virtual cluster, not in the host. This proves control plane isolation.


### References

(1*) https://www.vcluster.com/

(2*) [Workshop 1: Seamless Kubernetes Multi-Tenancy with vCluster and a Shared Platform Stack](https://www.youtube.com/live/HLCSbbQ_mQ8?si=x-vVRhSHFlHFQSSn&t=28)

(3*) [Solving Kubernetes Multi-tenancy Challenges with vCluster - By Fabian Brundke](https://liquidreply.net/news/solving-kubernetes-multi-tenancy-challenges-with-vcluster)
