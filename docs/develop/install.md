# Installation

----

## Preparation

- Install k8s or k3s. For the introduction of k8s, please refer to [kubernetes official website](https://kubernetes.io).

## Statement

- The k8s related informations used in this article are as follows:
```
// kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.6", GitCommit:"dff82dc0de47299ab66c83c626e08b245ab19037", GitTreeState:"clean", BuildDate:"2020-07-15T23:30:39Z", GoVersion:"go1.14.4", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.3", GitCommit:"2e7996e3e2712684bc73f0dec0200d64eec7fe40", GitTreeState:"clean", BuildDate:"2020-05-20T12:43:34Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```

- The baetyl-cloud related informations used in this article are as follows:
```
// git log
commit be8687f858b5c70d8dcdd55a120949fd64668588
```

Because the baetyl-cloud code is rapidly iterating, the latest code cannot be adapted in real time. So users need to switch to this version after downloading the baetyl-cloud code:
```shell script
git reset --hard be8687f858b5c70d8dcdd55a120949fd64668588
```
In addition, this article will be updated regularly to adapt to the latest baetyl-cloud code.

----

## Helm Quick Installation

This article supports using helm v2/v3 for installation. The relevant version information during testing is as follows:
```
// helm v3: helm version
version.BuildInfo{Version:"v3.2.3", GitCommit:"8f832046e258e2cb800894579b1b3b50c2d83492", GitTreeState:"clean", GoVersion:"go1.13.12"}

// helm v2: helm version
Client: &version.Version{SemVer:"v2.16.9", GitCommit:"8ad7037828e5a0fca1009dabe290130da6368e39", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.16.9", GitCommit:"8ad7037828e5a0fca1009dabe290130da6368e39", GitTreeState:"clean"}
```
For the installation of helm, please refer to [helm installation link](https://helm.sh/docs/intro/install).

### 1. Install database

Before installing baetyl-cloud, we need to install the database first, and execute the following command to install it.

```shell
// helm v3
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mariadb --set rootUser.password=secretpassword,db.name=baetyl_cloud bitnami/mariadb
helm install phpmyadmin bitnami/phpmyadmin 

// helm v2
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install --name mariadb --set rootUser.password=secretpassword,db.name=baetyl_cloud bitnami/mariadb
helm install --name phpmyadmin bitnami/phpmyadmin 
```
**Note**: For the convenience of demonstration, we have hardcoded the password, please modify it yourself, and you can replace secretpassword globally.

### 2. Initialize data

Confirm that mariadb and phpmyadmin are in the Running state.

```shell
kubectl get pod
# NAME                            READY   STATUS             RESTARTS   AGE
# mariadb-master-0                1/1     Running            0          2m56s
# mariadb-slave-0                 1/1     Running            0          2m56s
# phpmyadmin-55f4f964d7-ctmxj     1/1     Running            0          117s
```

Then execute the following command to keep the terminal from exiting.

```shell
export POD_NAME=$(kubectl get pods --namespace default -l "app=phpmyadmin,release=phpmyadmin" -o jsonpath="{.items[0].metadata.name}")
echo "phpMyAdmin URL: http://127.0.0.1:8080"
kubectl port-forward --namespace default svc/phpmyadmin 8080:80
```

Then use a browser to open http://127.0.0.1:8080/index.php, Server input: mariadb, Username input: root, Password input: secretpassword. After logging in, select the database baetyl_cloud, click the SQL button, and apply *scripts/common/tables.sql* and *scripts/native/sql/data.sql* under the baetyl-cloud project into the page for execution. If no error is reported during execution, the data initialization is successful. If you have installed before, please pay attention to delete the historical data under the baetyl-cloud database when installing again.

### 3. Install baetyl-cloud

For helm v3, enter the directory where the baetyl-cloud project is located and execute the following commands.

```shell
# helm v3
helm install baetyl-cloud ./scripts/charts/baetyl-cloud/
```

For helm v2, users need to do some additional operations in the above directories:

- Modify the apiVersion version in ./scripts/charts/baetyl-cloud/Chart.yaml from v2 to v1 and save it as follows:
```yaml
apiVersion: v1
name: baetyl-cloud
description: A Helm chart for Kubernetes

...
```
- Import crds manually:
```shell script
# k8s version is v1.16 or higher 
# k3s version is v1.17.4 or higher
kubectl apply -f ./scripts/charts/baetyl-cloud/apply/
# k8s version less than v1.16
# k3s version less than v1.17.4
kubectl apply -f ./scripts/charts/baetyl-cloud/apply_v1beta1/
```
- use helm v2 to install baetyl-cloud:
```shell script
# helm v2
helm install --name baetyl-cloud ./scripts/charts/baetyl-cloud/
```

To make sure that baetyl-cloud is in the Running state, and you can also check the log for errors.

```shell
kubectl get pod
# NAME                            READY   STATUS    RESTARTS   AGE
# baetyl-cloud-57cd9597bd-z62kb   1/1     Running   0          97s

kubectl logs -f baetyl-cloud-57cd9597bd-z62kb
```

### 4. Install edge node

Call the RESTful API to create a node.

```shell
curl -d "{\"name\":\"demo-node\"}" -H "Content-Type: application/json" -X POST http://0.0.0.0:30004/v1/nodes
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1931564","createTime":"2020-07-22T06:25:05Z","labels":{"baetyl-node-name":"demo-node"},"ready":false}
```

Obtain the online installation script of the edge node.

```shell
curl http://0.0.0.0:30004/v1/nodes/demo-node/init
# {"cmd":"sudo mkdir -p -m 666 /var/lib/baetyl/host /var/lib/baetyl/object /var/lib/baetyl/store /var/lib/baetyl/log /var/lib/baetyl/run && curl -skfL 'https://0.0.0.0:30003/v1/init/baetyl-init-deployment.yml?token=b98c8499f57b2265223a313630323831393239382c226e223a22313233222c226e73223a2262616574796c2d636c6f7564227d' -oinit.yml && kubectl delete clusterrolebinding baetyl-edge-system-rbac --ignore-not-found=true && kubectl delete ns baetyl-edge-system --ignore-not-found=true && kubectl apply -f init.yml"}
```

Execute the installation script on the machine where baetyl-cloud is deployed.

```shell
sudo mkdir -p -m 666 /var/lib/baetyl/host /var/lib/baetyl/object /var/lib/baetyl/store /var/lib/baetyl/log /var/lib/baetyl/run && curl -skfL 'https://0.0.0.0:30003/v1/init/baetyl-init-deployment.yml?token=b98c8499f57b2265223a313630323831393239382c226e223a22313233222c226e73223a2262616574796c2d636c6f7564227d' -oinit.yml && kubectl delete clusterrolebinding baetyl-edge-system-rbac --ignore-not-found=true && kubectl delete ns baetyl-edge-system --ignore-not-found=true && kubectl apply -f init.yml
```

**Note**: If you need to install an edge node on a device other than the machine where baetyl-cloud is deployed, please modify the database, change the sync-server-address and init-server-address in the baetyl_property  table to real addresses.

Check the status of the edge node. Eventually, two edge services will be in the Running state. You can also call the cloud RESTful API to view the edge node status. You can see that the edge node is online ("ready":true).

```shell
kubectl get pod -A
# NAMESPACE            NAME                                      READY   STATUS    RESTARTS   AGE
# baetyl-edge-system   baetyl-core-8668765797-4kt7r              1/1     Running   0          2m15s
# baetyl-edge-system   baetyl-function-5c5748957-nhn88           1/1     Running   0          114s

curl http://0.0.0.0:30004/v1/nodes/demo-node
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1939112",...,"report":{"time":"2020-07-22T07:25:27.495362661Z","sysapps":...,"node":...,"nodestats":...,"ready":true}
```

### 5. Uninstall baetyl-cloud

```shell
helm delete baetyl-cloud
# delete baetyl
kubectl delete ns baetyl-edge baetyl-edge-system
```

----

## K8S Installation

### 1. Install database

Install mysql database, and initialize the data as follows:

- Create 'baetyl_cloud' database and tables, see specific sql statements in: *scripts/common/tables.sql*

- Initialize table data, see specific sql setatements in: **scripts/k8s/sql/data.sql*

  ```shell
  # Note: modify the sync-server-address and init-server-address in baetyl_property to real server address：
  # For example, if the service is deployed locally, the address can be configured as follows：
  # sync-server-address : https://host ip:30005
  # init-server-address : https://0.0.0.0:30003
  # If the service is deployed on a non-local machine, please change IP to real server IP
  ```

- Modify the database configuration in *baetyl-cloud-configmap.yml*

  ```
  database:
  type: "mysql"
  url:"root:secretpassword@(host ip:3306)/baetyl_cloud?charset=utf8&parseTime=true"
  ```

  

### 2. Install baetyl-cloud

```shell
cd scripts/k8s
# k8s version is v1.16 or higher 
# k3s version is v1.17.4 or higher
kubectl apply -f ./apply/
# k8s version less than v1.16
# k3s version less than v1.17.4
kubectl apply -f ./apply_v1beta1/
```

After the execution is successful, you can execute `kubectl get pods |grep baetyl-cloud` command to check the program running status, and then you can operate the baetyl-cloud API via *http://0.0.0.0:30004*.
### 3. Install edge node

Call the RESTful API to create a node.

```shell
curl -d "{\"name\":\"demo-node\"}" -H "Content-Type: application/json" -X POST http://0.0.0.0:30004/v1/nodes
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1931564","createTime":"2020-07-22T06:25:05Z","labels":{"baetyl-node-name":"demo-node"},"ready":false}
```

Obtain the online installation script of the edge node.

```shell
curl http://0.0.0.0:30004/v1/nodes/demo-node/init
# {"cmd":"sudo mkdir -p -m 666 /var/lib/baetyl/host /var/lib/baetyl/object /var/lib/baetyl/store /var/lib/baetyl/log /var/lib/baetyl/run && curl -skfL 'https://0.0.0.0:30003/v1/init/baetyl-init-deployment.yml?token=b98c8499f57b2265223a313630323831393239382c226e223a22313233222c226e73223a2262616574796c2d636c6f7564227d' -oinit.yml && kubectl delete clusterrolebinding baetyl-edge-system-rbac --ignore-not-found=true && kubectl delete ns baetyl-edge-system --ignore-not-found=true && kubectl apply -f init.yml"}
```

Execute the installation script on the machine where baetyl-cloud is deployed.

```shell
sudo mkdir -p -m 666 /var/lib/baetyl/host /var/lib/baetyl/object /var/lib/baetyl/store /var/lib/baetyl/log /var/lib/baetyl/run && curl -skfL 'https://0.0.0.0:30003/v1/init/baetyl-init-deployment.yml?token=b98c8499f57b2265223a313630323831393239382c226e223a22313233222c226e73223a2262616574796c2d636c6f7564227d' -oinit.yml && kubectl delete clusterrolebinding baetyl-edge-system-rbac --ignore-not-found=true && kubectl delete ns baetyl-edge-system --ignore-not-found=true && kubectl apply -f init.yml
```

**Note**: If you need to install an edge node on a device other than the machine where baetyl-cloud is deployed, please modify the database, change the  sync-server-address and init-server-address  in the baetyl_property table to real addresses.

Check the status of the edge node. Eventually, two edge services will be in the Running state. You can also call the cloud RESTful API to view the edge node status. You can see that the edge node is online ("ready":true).

```shell
kubectl get pod -A
# NAMESPACE            NAME                                      READY   STATUS    RESTARTS   AGE
# baetyl-edge-system   baetyl-core-8668765797-4kt7r              1/1     Running   0          2m15s
# baetyl-edge-system   baetyl-function-5c5748957-nhn88           1/1     Running   0          114s

curl http://0.0.0.0:30004/v1/nodes/demo-node
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1939112",...,"report":{"time":"2020-07-22T07:25:27.495362661Z","sysapps":...,"node":...,"nodestats":...,"ready":true}
```

### 4. Uninstall baetyl-cloud

```shell
cd scripts/k8s
# k8s version is v1.16 or higher 
# k3s version is v1.17.4 or higher
kubectl delete -f ./apply/
# k8s version less than v1.16
# k3s version less than v1.17.4
kubectl delete -f ./apply_v1beta1/
# delete baetyl
kubectl delete ns baetyl-edge baetyl-edge-system
```

----

## Process installation

### 1. Install database

Install mysql database, and initialize the data as follows:

- Create 'baetyl_cloud' database and tables, see specific sql statement in: *scripts/common/tables.sql*

- Initialize table data, see specific sql statement in : *scripts/native/sql/data.sql*

  ```shell
  # Note: modify the sync-server-address and init-server-address in baetyl_property to real server address：
  # For example, if the service is deployed locally, the address can be configured as follows：
  # sync-server-address : https://host ip:30005
  # init-server-address : https://0.0.0.0:30003
  # If the service is deployed on a non-local machine, please change IP to real server IP
  ```

- Modify the database configuration in *conf/conf.yml*

  ```
  database:
  type: "mysql"
  url:"root:secretpassword@(localhost:3306)/baetyl_cloud?charset=utf8&parseTime=true"
  ```

  

### 2. Source code compilation

Refer [Source code compilation](../develop/build.md)

### 3. Install baetyl-cloud

```shell
# Import k8s crd resources
cd scripts/native

# k8s version is v1.16 or higher 
# k3s version is v1.17.4 or higher
kubectl apply -f ./apply/
# k8s version less than v1.16
# k3s version less than v1.17.4
kubectl apply -f ./apply_v1beta1/

# Execute the following command, replace example in the conf/kubeconfig.yml file
kubectl config view --raw
# Execute the following command
nohup ../../../output/baetyl-cloud -c ./conf/conf.yml > /dev/null &
# After successful execution, it will return the successfully established baetyl-cloud process number
```

After successful execution, you can operate baetyl-cloud API via *http://0.0.0.0:9004*.
### 4. Install edge node

Call the RESTful API to create a node.

```shell
curl -d "{\"name\":\"demo-node\"}" -H "Content-Type: application/json" -X POST http://0.0.0.0:9004/v1/nodes
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1931564","createTime":"2020-07-22T06:25:05Z","labels":{"baetyl-node-name":"demo-node"},"ready":false}
```

Obtain the online installation script of the edge node.

```shell
curl http://0.0.0.0:9004/v1/nodes/demo-node/init
# {"cmd":"sudo mkdir -p -m 666 /var/lib/baetyl/host /var/lib/baetyl/object /var/lib/baetyl/store /var/lib/baetyl/log /var/lib/baetyl/run && curl -skfL 'https://0.0.0.0:9003/v1/init/baetyl-init-deployment.yml?token=b98c8499f57b2265223a313630323831393239382c226e223a22313233222c226e73223a2262616574796c2d636c6f7564227d' -oinit.yml && kubectl delete clusterrolebinding baetyl-edge-system-rbac --ignore-not-found=true && kubectl delete ns baetyl-edge-system --ignore-not-found=true && kubectl apply -f init.yml"}
```

Execute the installation script on the machine where baetyl-cloud is deployed.

```shell
sudo mkdir -p -m 666 /var/lib/baetyl/host /var/lib/baetyl/object /var/lib/baetyl/store /var/lib/baetyl/log /var/lib/baetyl/run && curl -skfL 'https://0.0.0.0:9003/v1/init/baetyl-init-deployment.yml?token=b98c8499f57b2265223a313630323831393239382c226e223a22313233222c226e73223a2262616574796c2d636c6f7564227d' -oinit.yml && kubectl delete clusterrolebinding baetyl-edge-system-rbac --ignore-not-found=true && kubectl delete ns baetyl-edge-system --ignore-not-found=true && kubectl apply -f init.yml
```

**Note**: If you need to install an edge node on a device other than the machine where baetyl-cloud is deployed, please modify the database, change the sync-server-address  and  init-server-address in the baetyl_property table to real addresses.

Check the status of the edge node. Eventually, two edge services will be in the Running state. You can also call the cloud RESTful API to view the edge node status. You can see that the edge node is online ("ready":true).

```shell
kubectl get pod -A
# NAMESPACE            NAME                                      READY   STATUS    RESTARTS   AGE
# baetyl-edge-system   baetyl-core-8668765797-4kt7r              1/1     Running   0          2m15s
# baetyl-edge-system   baetyl-function-5c5748957-nhn88           1/1     Running   0          114s

curl http://0.0.0.0:9004/v1/nodes/demo-node
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1939112",...,"report":{"time":"2020-07-22T07:25:27.495362661Z","sysapps":...,"node":...,"nodestats":...,"ready":true}
```

### 5. Uninstall baetyl-cloud

```shell
# Kill the process according to the process number when the creation is successful:
sudo kill process number
# delete baetyl
kubectl delete ns baetyl-edge baetyl-edge-system
```

