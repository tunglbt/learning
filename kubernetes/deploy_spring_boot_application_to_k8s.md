# Triển khai ứng dụng Spring Boot lên Kubernetes


## Triển khai lần đầu Spring Boot application lên K8s

### 1. Dockerfile
Tạo file Dockerfile trong thư mục của project (cùng cấp với file `pom.xml`)
```dockerfile
FROM openjdk:8-jre-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

### 2. Login vào Gitlab Registry
Dùng username và token của Gitlab để login vào Gitlab Registry (chỉ chạy một lần).
```shell
docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
```

### 3. Build Docker image và push lên Gitlab Registry
```shell
mvn clean install -P <profile> -DskipTests
docker build -t <registry-path>/<image-name>:<tag> . --platform=linux/amd64
docker push <registry-path>/<image-name>:<tag>
```

### 4. Tạo Deployment
Tạo file `deployment.yml` có cấu trúc như sau:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <deployment-name>
  namespace: <namespace>
spec:
  replicas: 1
  selector:
    matchLabels:
      app: <app-name>
  template:
    metadata:
      name: <app-name>
      labels:
        app: <app-name>
    spec:
      containers:
        - name: <app-name>
          image: <registry-path>/<image-name>:<tag>
          imagePullPolicy: Always
          ports:
            - containerPort: <app-port>
      imagePullSecrets: # cần tạo registry-credentials để pull image từ registry
        - name: registry-credentials 
```
Áp dụng các thiết lập trên để tạo deployment trên K8s:
```shell
kubectl apply -f deployment.yml
```

## Cập nhật image mới cho deployment trên K8s

### 1. Build Docker image và push lên Gitlab Registry
```shell
mvn clean install -P <profile> -DskipTests
docker build -t <registry-path>/<image-name>:<tag> . --platform=linux/amd64
docker push <registry-path>/<image-name>:<tag>
```

### 2. Restart deployment 
```shell
kubectl -n <namespace> rollout restart deployment <deployment-name>
```

## Tham khảo
- [GitLab Container Registry](https://docs.gitlab.com/ee/user/packages/container_registry/)