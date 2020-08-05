# Baetyl-cloud installation

----

## Preparation

- Install k8s/k3s

----

## Helm Quick Installation

### 1. Install database

Before installing baetyl-cloud, we need to install the database first, and execute the following command to install it.

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mariadb --set rootUser.password=secretpassword,db.name=baetyl_cloud bitnami/mariadb
helm install phpmyadmin bitnami/phpmyadmin 
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

Then use a browser to open http://127.0.0.1:8080/index.php, Server input: mariadb, Username input: root, Password input: secretpassword. After logging in, select the database baetyl_cloud, click the SQL button, and enter the sql statements of all files in the scripts/sql directory under the baetyl-cloud project into the page for execution. If no error is reported during execution, the data initialization is successful.

### 3. Install baetyl-cloud

Enter the directory where the baetyl-cloud project is located and execute the following commands.

```shell
# helm 3
helm install baetyl-cloud ./scripts/demo/charts/baetyl-cloud/
```
Make sure that baetyl-cloud is in the Running state, and you can also check the log for errors.

```shell
kubectl get pod
# NAME                            READY   STATUS    RESTARTS   AGE
# baetyl-cloud-57cd9597bd-z62kb   1/1     Running   0          97s

kubectl logs -f baetyl-cloud-57cd9597bd-z62kb
```

### 4. Create and install edge node

Call the RESTful API to create a node.

```shell
curl -d "{\"name\":\"demo-node\"}" -H "Content-Type: application/json" -X POST http://0.0.0.0:30004/v1/nodes
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1931564","createTime":"2020-07-22T06:25:05Z","labels":{"baetyl-node-name":"demo-node"},"ready":false}
```

Obtain the online installation script of the edge node.

```shell
curl http://0.0.0.0:30004/v1/nodes/demo-node/init
# {"cmd":"curl -skfL 'https://0.0.0.0:30003/v1/active/setup.sh?token=f6d21baa9b7b2265223a333630302c226b223a226e6f6465222c226e223a2264656d6f2d6e6f6465222c226e73223a2262616574796c2d636c6f7564222c227473223a313539353430323132367d' -osetup.sh && sh setup.sh"}
```

Execute the installation script on the machine where baetyl-cloud is deployed.

```shell
curl -skfL 'https://0.0.0.0:30003/v1/active/setup.sh?token=f6d21baa9b7b2265223a333630302c226b223a226e6f6465222c226e223a2264656d6f2d6e6f6465222c226e73223a2262616574796c2d636c6f7564222c227473223a313539353430323132367d' -osetup.sh && sh setup.sh
```

**Note**: If you need to install an edge node on a device other than the machine where baetyl-cloud is deployed, please modify the database, change the node-address and active-address in the baetyl_system_config table to real addresses.

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
```

----

## K8s Installation

### 1. Install database

Install mysql database, and initialize the data as follows:

- Create 'baetyl-cloud' database and tables, see specific sql statements  in : *scripts/sql/tables.sql*

- Initialize table data, see specific sql statements in : *scripts/sql/data.sql*

  ```shell
  # Note: modify the node-address and active-address in baetyl_system_config to the actual server address：
  # For example, if the service is deployed locally, the address can be configured as follows：
  # node-address : https://0.0.0.0:30005
  # active-address : https://0.0.0.0:30003
  # If the service is deployed on a non-local machine, please change the IP to the actual server IP address
  ```

- Modify the database configuration in *baetyl-cloud-configmap.yml*

### 2. Install baetyl-cloud

```shell
cd scripts/demo/k8s
kubectl apply -f .
```

After the execution is successful, you can execute `kubectl get pods |grep baetyl-cloud` command to check the program running status, and then you can operate the baetyl-cloud API via *http://0.0.0.0:30004*.
### 3. Create and install edge node

Call the RESTful API to create a node.

```shell
curl -d "{\"name\":\"demo-node\"}" -H "Content-Type: application/json" -X POST http://0.0.0.0:30004/v1/nodes
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1931564","createTime":"2020-07-22T06:25:05Z","labels":{"baetyl-node-name":"demo-node"},"ready":false}
```

Obtain the online installation script of the edge node.

```shell
curl http://0.0.0.0:30004/v1/nodes/demo-node/init
# {"cmd":"curl -skfL 'https://0.0.0.0:30003/v1/active/setup.sh?token=f6d21baa9b7b2265223a333630302c226b223a226e6f6465222c226e223a2264656d6f2d6e6f6465222c226e73223a2262616574796c2d636c6f7564222c227473223a313539353430323132367d' -osetup.sh && sh setup.sh"}
```

Execute the installation script on the machine where baetyl-cloud is deployed.

```shell
curl -skfL 'https://0.0.0.0:30003/v1/active/setup.sh?token=f6d21baa9b7b2265223a333630302c226b223a226e6f6465222c226e223a2264656d6f2d6e6f6465222c226e73223a2262616574796c2d636c6f7564222c227473223a313539353430323132367d' -osetup.sh && sh setup.sh
```

**Note**: If you need to install an edge node on a device other than the machine where baetyl-cloud is deployed, please modify the database, change the node-address and active-address in the baetyl_system_config table to real addresses.

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
cd scripts/demo/k8s
kubectl delete -f .
```

----

## Process installation

### 1. Install database

Install mysql database, and initialize the data as follows:

- Create 'baetyl-cloud' database and tables, see specific sql statement  in : *scripts/sql/tables.sql*

- Initialize table data, see specific sql statement in : *scripts/sql/data.sql*

  ```shell
  # Note: modify the node-address and active-address in baetyl_system_config to the actual server address：
  # For example, if the service is deployed locally, the address can be configured as follows：
  # node-address : https://0.0.0.0:30005
  # active-address : https://0.0.0.0:30003
  # If the service is deployed on a non-local machine, please change the IP to the actual server IP address
  ```

- Modify the database configuration in *conf/cloud.yml*

### 2. Source code compilation

Refer [Source code compilation](../develop/build.html)

### 3. Install baetyl-cloud

```shell
# Import k8s crd resources
kubectl apply -f scripts/demo/native/conf/crds.yml
cd scripts/demo/native
# Execute the following command, replace example in the conf/kubeconfig.yml file
kubectl config view --raw
# Execute the following command
nohup ../../../output/baetyl-cloud -c ./conf/cloud.yml > /dev/null &
# After successful execution, it will return the successfully established baetyl-cloud process number
```

After successful execution, you can operate baetyl-cloud API via *http://0.0.0.0:9004*.
### 4. Create and install edge node

Call the RESTful API to create a node.

```shell
curl -d "{\"name\":\"demo-node\"}" -H "Content-Type: application/json" -X POST http://0.0.0.0:30004/v1/nodes
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1931564","createTime":"2020-07-22T06:25:05Z","labels":{"baetyl-node-name":"demo-node"},"ready":false}
```

Obtain the online installation script of the edge node.

```shell
curl http://0.0.0.0:30004/v1/nodes/demo-node/init
# {"cmd":"curl -skfL 'https://0.0.0.0:30003/v1/active/setup.sh?token=f6d21baa9b7b2265223a333630302c226b223a226e6f6465222c226e223a2264656d6f2d6e6f6465222c226e73223a2262616574796c2d636c6f7564222c227473223a313539353430323132367d' -osetup.sh && sh setup.sh"}
```

Execute the installation script on the machine where baetyl-cloud is deployed.

```shell
curl -skfL 'https://0.0.0.0:30003/v1/active/setup.sh?token=f6d21baa9b7b2265223a333630302c226b223a226e6f6465222c226e223a2264656d6f2d6e6f6465222c226e73223a2262616574796c2d636c6f7564222c227473223a313539353430323132367d' -osetup.sh && sh setup.sh
```

**Note**: If you need to install an edge node on a device other than the machine where baetyl-cloud is deployed, please modify the database, change the node-address and active-address in the baetyl_system_config table to real addresses.

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
# Kill the process according to the process number when the creation is successful:
sudo kill process number
```

