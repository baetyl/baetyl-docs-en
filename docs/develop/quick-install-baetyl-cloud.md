# Install from source

quick installation of baetyl-cloud, you can build baetyl-cloud from source to get the latest features.

## Prerequisites

- The Go tools and modules

The minimum required go version is 1.13. Refer to [golang.org](https://golang.org/dl/) or [golang.google.cn](https://golang.google.cn/dl/) to download and install the Go tools. Now we use Go Modules to manage packages，you can refer [goproxy.baidu.com](https://goproxy.baidu.com/)  to set GOPROXY if needs.

- The Docker Engine and Buildx

The minimum required Docker version is 19.03, because the Docker Buildx feature is introduced to build multi-platform images. Refer to  [docker.com/install](https://docs.docker.com/install/)to install the Docker Engine and refer to [github.com/docker/buildx](https://github.com/docker/buildx) to enable the Docker Buildx.


-Install the database

This database mysql must pre-installed, [mysql download](https://dev.mysql.com/downloads/mysql/).

-Install k8s/k3s

This k3s must pre-installed.

```shell
curl -sfL https://get.k3s.io | sh-
```
also can use:
```shell
curl -sfL https://docs.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh-
```

-Install helm

See [helm official website](https://helm.sh/)

```shell
brew install helm
```

## Download the baetyl-cloud source code

Download the code from [Baetyl Github](https://github.com/baetyl/baetyl-cloud) and execute the following command:

```shell
git clone https://github.com/baetyl/baetyl-cloud.git
```

## Compile baetyl-cloud

Go into the baetyl-cloud project directory and execute `make` to compile the baetyl-cloud main program.

```shell
make
```

After the make command is completed, the baetyl-cloud main program will be generated in the project's `output` directory.

## baetyl-cloud configuration

Please refer to [Configuration Interpretation](./baetyl-cloud-config-interpretation.md) for baetyl-cloud configuration definition description.
Refer to the configuration interpretation to make related settings. The cloud.yml reference is as follows:
 ```shell
activeServer:
  port: ":9003"
  ca: "./client_ca.crt"
  cert: "./server.crt"
  key: "./server.key"

adminServer:
  port: ":9004"

nodeServer:
  port: ":9005"
  ca: "./server_ca.crt"
  cert: "./server.crt"
  key: "./server.key"

plugin:
  auth: "defaultauth"
  license: "defaultlicense"
  pki: "defaultpki"
  shadow: "database"
  modelStorage: "kubernetes"
  databaseStorage: "database"

logger:
  filename: "./baetyl/baetyl-cloud.log"
  level: debug

kubernetes:
  inCluster: false
  configPath: "./kubernetes.yml" #need to be set kubernetes configuration in preparation

database:
  type: "mysql"
  url: "username:password@(localhost:port)/baetyl_cloud?charset=utf8&parseTime=true" #need to change to the database address in preparation

defaultpki:
  rootCAFile: "./client_ca.crt"
  rootCAKeyFile: "./client_ca.key"
  persistent:
    kind: "database"
    database:
      type: "mysql"
      url: "username:password@(localhost:port)/baetyl_cloud?charset=utf8&parseTime=true" #Need to change to the database address in preparation
       
defaultauth:
  namespace: "baetyl-cloud"
  keyFile: "./token.key"
```


## Create baetyl-cloud CRD in kubernetes

Set the k8s cluster configuration and create the CRD file in scripts/crd/crd.yml.

```shell
kubectl apply -f scripts/crd/crd.yml
```

## Database initialization

Create the baetyl-cloud database and tables, see scripts/sql/tables.sql
Initialize the datas, see examples/sql/data.sql

## Create certificate files

Use the openssl to generate server_ca.crt, server.key, server.crt , client_ca.crt, client_ca.key in the `output` directory, and generate token.key by use RSA algorithm.

You can refer to the certificate files in examples/charts/baetyl-cloud/certs/  and the examples/charts/baetyl-cloud/conf/token.key file.

## Run baetyl-cloud

There are three ways to run the baetyl-cloud.

### 1. Deploy the baetyl-cloud in k8s

Modify the database's configurations in examples/k8s/baetyl-cloud-configmap.yml, and run the following commands:
```shell
kubectl apply -f baetyl-cloud-configmap.yml
kubectl apply -f baetyl-cloud-rabc.yml
kubectl apply -f baetyl-cloud-deployment.yml
```
Then you can through the `kubectl get pods | grep baetyl-cloud` command to see baetyl-cloud run status. If baetyl-cloud is running, you can access the API through http://127.0.0.1:9004. The API is [here](./api.md).

### 2. Start baetyl-cloud in mirror mode

* Replace the database address in examples/charts/baetyl-cloud/conf/cloud.yml with database's address in preparation;
* Replace examples/charts/baetyl-cloud/conf/k8s.yml with k8s/k3s configuration;
* Copy the examples/charts/baetyl-cloud/ directory to the target deployed machine, and execute `helm install baetyl-cloud`. to install;
* You can see the running status of the program through the `kubectl get pods |grep baetyl-cloud` command;
* You can access the API through http://0.0.0.0:30004. The API is [here](./api.md).

### 3. Run baetyl-cloud in process mode

Run the baetyl-cloud by the following commands:

```shell
cd output
nohup ./baetyl-cloud -c ./cloud.yml> /dev/null &
```
After started, you can access the API through http://127.0.0.1:9004. The API is [here](./api.md).
 
## Make a mirror
 
If you use container mode to run baetyl-cloud, we recommend using officially released official images. If you want to make your own image, you can use the commands provided below, but only if the Buildx function mentioned in the first preparation is turned on.

Go into the baetyl-cloud project directory and execute `make image` to generate baetyl-cloud image.

```shell
make image
```

After the command is completed, you can execute `docker images` to view the generated image.

```shell
docker images

REPOSITORY    TAG          IMAGE ID         CREATED SIZE
cloud       git-be2c5a9   d70a7faf5443    About an hour ago 40.7MB
```
Modify the image configuration in examples/charts/baetyl-cloud/values.yaml, and use helm install command to deploy baetyl-cloud.