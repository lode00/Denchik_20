## Подготовка окружения и соединений
```markdown
## Генерируем ssh-ключ для связи gitlab и нашего сервера
root@project-devops:~# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:crnx8g8o8d3KVmnyB06qaiJfesN4obbgNgAXjtJ1VH8 root@project-devops
The key's randomart image is:
+---[RSA 3072]----+
|     ....        |
|  . . .  .       |
| + o .    . E    |
|+ +      . .     |
|o.    o S    .   |
|.     .= *..*    |
| ..  +o.= +O..   |
| .+.=o*. +ooo .  |
| ..=+*.o.o+...   |
+----[SHA256]-----+
```
```markdown
## Проверяем, что ключи добавились и копируем его
root@project-devops:~# cat /root/.ssh/id_rsa.pub  
``` 
```markdown
## клонируем наш проект
root@project-devops:~# git clone git@gitlab.com:lobachdenis00/course-work.git
```
```markdown
## Инициализируем наш проект
root@project-devops::~/course-work# git init .
```
```markdown
## Коммитим
root@project-devops:~/course-work# git commit -am 'Add app'
```
```markdown
## Пушим в gitlab
root@project-devops:~/course-work# git push origin main
```
## Добавляем gitlab-runner
```bash
root@project-devops:~#  curl -L --output /usr/local/bin/gitlab-runner   https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
root@project-devops:~#  chmod +x /usr/local/bin/gitlab-runner
root@project-devops:~#  sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
root@project-devops:~# sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
Runtime platform                                    arch=amd64 os=linux pid=1946 revision=98daeee0 version=14.7.0
root@project-devops:~# sudo gitlab-runner start
Runtime platform                                    arch=amd64 os=linux pid=2008 revision=98daeee0 version=14.7.0
```
```bash
root@project-devops:~# sudo gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=2035 revision=98daeee0 version=14.7.0
Running in system-mode.
Enter the GitLab instance URL (for example, https://gitlab.com/):
https://gitlab.com/
Enter the registration token:
16mfPr9zrEyqaqHkz7JF
Enter a description for the runner:
[project-devops]: project-devops
Enter tags for the runner (comma-separated):
docker
Registering runner... succeeded                     runner=16mfPr9z
Enter an executor: custom, docker-ssh, ssh, kubernetes, docker, parallels, shell, virtualbox, docker+machine, docker-ssh+machine:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```
## docker-compose.yaml для локальной сборки
```bash
version: '3'
services:
  postgres:
    restart: always
    container_name: postgres
    image: postgres:13
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'example'
      POSTGRES_DB: 'dev-school'
    ports:
      - '5432:5432'
  back:
    restart: always
    container_name: backend
    image: dev-school-app:1.0-SNAPSHOT
    environment:
      DATASOURCE_URL: jdbc:postgresql://postgres:5432/dev-school
      DATASOURCE_USERNAME: postgres
      DATASOURCE_PASSWORD: example
      BASE_PATH: http://localhost:8080
    ports:
      - '8080:8080'
    depends_on:
      - postgres
  front:
    restart: always
    image: devschool-front-app:1.0.0
    container_name: frontend
    ports:
      - '8081:8081'
    depends_on:
      - back
```
```markdown
## Gitlab registry 
root@project-devops:~/course-work/dev-school-app#  docker tag 0fccd173f06c  registry.gitlab.com/lobachdenis00/course-work/devschool-front-app:1.0.0
root@project-devops:~/course-work/dev-school-app# docker login registry.gitlab.com -u lobachdenis00 -p glpat-3HyahgtB9gJb9xpvd78b
root@project-devops:~/course-work/dev-school-app# docker push   registry.gitlab.com/lobachdenis00/course-work/devschool-front-app:1.0.0
```
## Предварительная настройка gitlab-runner
```markdown
## docker установка
curl -fsSL https://get.docker.com -o get-docker.sh 
sudo sh get-docker.sh 
sudo chmod 666 /var/run/docker.sock
## java+node+yarn
apt-get update
apt-get install openjdk-8-jdk -y
apt-get install -y curl vim wget gcc g++ make
curl -fsSL https://deb.nodesource.com/setup_14.x | bash -
apt-get install -y nodejs
wget "https://github.com/yarnpkg/yarn/releases/download/v1.22.11/yarn_1.22.11_all.deb"
dpkg -i yarn_1.22.11_all.deb 
rm yarn_1.22.11_all.deb
## docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## certificate-production.yaml
```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: ssl-cert-production
  namespace: dlobach-prod
spec:
  secretName: ssl-cert-production
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  commonName: dlobach-prod.school.telekom.sh
  dnsNames:
  - dlobach-prod.school.telekom.sh
  
 ``` 
## production-issuer.yaml 
 ```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-production
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: lobachdenis00@gmail.com
    privateKeySecretRef:
      name: letsencrypt-production
    solvers:
    - selector: {}
      http01:
        ingress:
          class: nginx
```
## Dockerfile 
``` 

FROM alpine

# Ignore to update versions here
# docker build --no-cache --build-arg KUBECTL_VERSION=${tag} --build-arg HELM_VERSION=${helm} --build-arg KUSTOMIZE_VERSION=${kustomize_version} -t ${image}:${tag} .
ARG HELM_VERSION=3.2.1
ARG KUBECTL_VERSION=1.17.5
ARG KUSTOMIZE_VERSION=v3.8.1
ARG KUBESEAL_VERSION=v0.15.0

# Install helm (latest release)
# ENV BASE_URL="https://storage.googleapis.com/kubernetes-helm"
ENV BASE_URL="https://get.helm.sh"
ENV TAR_FILE="helm-v${HELM_VERSION}-linux-amd64.tar.gz"
RUN apk add --update --no-cache curl ca-certificates bash git && \
    curl -sL ${BASE_URL}/${TAR_FILE} | tar -xvz && \
    mv linux-amd64/helm /usr/bin/helm && \
    chmod +x /usr/bin/helm && \
    rm -rf linux-amd64
# Install awscli
RUN apk add --update --no-cache python3 && \
    python3 -m ensurepip && \
    pip3 install --upgrade pip && \
    pip3 install awscli && \
    pip3 cache purge
# Install kubectl (same version of aws esk)
RUN curl -sLO https://storage.googleapis.com/kubernetes-release/release/v${KUBECTL_VERSION}/bin/linux/amd64/kubectl && \
    mv kubectl /usr/bin/kubectl && \
    chmod +x /usr/bin/kubectl
RUN mkdir /home/.kube/

ADD config /home/.kube/
RUN mkdir ~/.kube/
RUN cp /home/.kube/config ~/.kube/
EXPOSE 8080

 ```
  
  

