## Native Java ビルド環境
* Download and install the GraalVM JDK in the folder with the following commands:
```
mkdir -p <MY_FOLDER_FULL_PATH>
cd <MY_FOLDER_FULL_PATH>

graalvm_version=21.1.0
graalvm_archive=graalvm-ce-java11-linux-amd64-${graalvm_version}
graalvm_folder=graalvm-ce-java11-${graalvm_version}
curl -L https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-${graalvm_version}/${graalvm_archive}.tar.gz > ${graalvm_archive}.tar.gz
tar -xvf ${graalvm_archive}.tar.gz
rm ${graalvm_archive}.tar.gz
```
* To be able to install the GraalVM native compiler, the environment variables JAVA_HOME and PATH need to be configured as:
```
export JAVA_HOME=$PWD/${graalvm_folder}
export PATH=$JAVA_HOME/bin:$PATH
```
* Install the GraalVM native image compiler, including its tracing agent, with the GraalVM updater program, gu, and check its version number with the following commands:
```
gu install native-image
native-image --version

> GraalVM 21.1.0 Java 11 CE ...
```
* Configure the Bash startup file for GraalVM and its tracing agent.
```
export graalvm_version=21.1.0
export graalvm_folder=graalvm-ce-java11-${graalvm_version}
export JAVA_HOME=<MY_FOLDER_FULL_
```

* Start a new Terminal window, run the command java -version.
```
> openjdk version "11.0.11" 2021-04-20
> OpenJDK Runtime Environment GraalVM CE 21.1.0 (build 11.0.11+8-jvmci-21.1-b05)
> OpenJDK 64-Bit Server VM GraalVM CE 21.1.0 (build 11.0.11+8-jvmci-21.1-b05, mixed mode, sharing)
```

## ビルド手順
```
./gradlew :microservices:product-service:bootBuildImage --no-daemon
./gradlew :microservices:product-composite-service:bootBuildImage --no-daemon
./gradlew :microservices:recommendation-service:bootBuildImage --no-daemon
./gradlew :microservices:review-service:bootBuildImage --no-daemon
```
* かなり時間がかかります
```
minikube start \
 --profile=handson-spring-boot-cloud \
 --memory=10240 \
 --cpus=16 \
 --disk-size=30g \
 --kubernetes-version=v1.24.1 \
 --driver=docker \
 --ports=8080:80 --ports=8443:443 \
 --ports=9411:9411 --ports=30080:30080 --ports=30443:30443 
```
```
minikube profile handson-spring-boot-cloud
minikube addons enable metrics-server
minikube addons enable ingress
```
```
 helm upgrade --install cert-manager jetstack/cert-manager \
--create-namespace \
--namespace cert-manager \
--version v1.11.0 \
--set installCRDs=true \
--wait
```

```
docker save hands-on/native-product-composite-service:latest -o native-product-composite.tar
docker save hands-on/native-product-service:latest -o native-product.tar
docker save hands-on/native-recommendation-service:latest -o native-recommendation.tar
docker save hands-on/native-review-service:latest -o native-review.tar
```
```
eval $(minikube docker-env)
docker load -i native-product-composite.tar
docker load -i native-product.tar
docker load -i native-recommendation.tar
docker load -i native-review.tar

rm native-product-composite.tar native-product.tar native-recommendation.tar native-review.tar
```
```
docker-compose build auth-server

docker pull zookeeper:3.4.14
docker pull wurstmeister/kafka:2.12-2.5.0
```
```
kubectl delete namespace hands-on
kubectl apply -f kubernetes/hands-on-namespace.yml
kubectl config set-context $(kubectl config current-context) --namespace=hands-on

for f in kubernetes/helm/components/*; do helm dep up $f; done
for f in kubernetes/helm/environments/*; do helm dep up $f; done

helm upgrade --install hands-on-dev-env-native  kubernetes/helm/environments/dev-env-native  -n hands-on --wait
```
