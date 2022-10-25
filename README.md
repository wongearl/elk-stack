## 1. 创建secret

### 1.1 创建ES集群加密证书
1. 运行容器生成证书
```shell
docker run --name elastic-charts-certs -i -w /app elasticsearch:7.13.1 /bin/sh -c \
"elasticsearch-certutil ca --days 3650 --out /app/elastic-stack-ca.p12 --pass '' && \
elasticsearch-certutil cert --name security-master --dns \
security-master --ca /app/elastic-stack-ca.p12 --pass '' --ca-pass '' --out /app/elastic-certificates.p12"
```
2. 复制生成的证书到本地
```shell
docker cp elastic-charts-certs:/app/elastic-certificates.p12 ./
```
3. 删除容器
```shell
docker rm -f elastic-charts-certs
```
4. 将 pcks12 中的信息分离出来，写入文件
```shell
openssl pkcs12 -nodes -passin pass:'' -in elastic-certificates.p12 -out elastic-certificate.pem
```
### 1.2 使用证书创建secret
1. 创建 namespace
```shell
kubectl create namespace elkstuck
```
2. 创建 ELK 集群 SSL（p12） 证书的证书
```shell
kubectl -n elkstuck create secret generic elastic-certificates --from-file=./elastic-certificates.p12
kubectl -n elkstuck create secret generic elastic-certificate-pem --from-file=./elastic-certificate.pem
```
3. 创建 ELK 集群的管理员账号密码,用户名使用 `elastic`
```shell
kubectl -n elkstuck create secret generic elastic-credentials --from-literal=username=elastic --from-literal=password=elastic
```

