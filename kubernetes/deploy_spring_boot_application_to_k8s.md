# Triển khai ứng dụng Spring Boot lên Kubernetes

## 1. Dockerfile
Tạo file Dockerfile trong thư mục của project (cùng cấp với file `pom.xml`)
```dockerfile
FROM openjdk:8-jre-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```
## 2. Login vào Gitlab Registry
Dùng username và token của Gitlab để login vào Gitlab Registry (chỉ chạy một lần).
```shell
docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
```

## 3. Build Docker image và push lên Gitlab Registry
```shell
mvn clean install -P <profile> -DskipTests
docker build -t <registry-path>/<image-name>:<tag> . --platform=linux/amd64
docker push <registry-path>/<image-name>:<tag>
```
## 4. Restart deployment 
```shell
kubectl -n <namespace> rollout restart deployment <deployment-name>
```