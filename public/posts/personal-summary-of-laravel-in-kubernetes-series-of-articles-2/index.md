# Laravel in Kubernetes 系列文章个人总结（2）


## 前言

本篇我们继续总结 [https://chris-vermeulen.com/laravel-in-kubernetes/](https://chris-vermeulen.com/laravel-in-kubernetes/) 系列文章，上一篇我们总结完毕了 docker-composer 本地测试部分。后面还有几篇是集群设置的，这个我就没有总结，因为他的设置有平台依赖。那么在真实使用中，我们可以用公有云服务，到时候我们只要弄后面总计的配置就好了，或者自己搭建集群。
单节点测试集群可以看我之前的文章
[https://www.cimple.ink/2022/04/06/ubuntu-install-k8s-and-use-metallb-with-nginx-ingerss/](https://www.cimple.ink/2022/04/06/ubuntu-install-k8s-and-use-metallb-with-nginx-ingerss/)
[https://www.cimple.ink/2022/04/17/ubuntu-k8s-stateful-mysql/](https://www.cimple.ink/2022/04/17/ubuntu-k8s-stateful-mysql/)。

## 正文

### 创建配置文件目录
```bash
mkdir -p laravel-in-kubernetes-deployment
cd laravel-in-kubernetes-deployment
```
我们所有的配置文件都会保存在这里，不得不说他的这个目录规划也是很棒，让我学到了很多东西。

### ConfigMap And Secret
创建 `common` 目录，保存通用信息。

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: laravel-in-kubernetes
data:
  APP_NAME: "Laravel"
  APP_ENV: "local"
  APP_DEBUG: "true"
  # Once you have an external URL for your application, you can add it here. 
  APP_URL: "http://laravel-in-kubernetes.test"
  
  # Update the LOG_CHANNEL to stdout for Kubernetes
  LOG_CHANNEL: "stdout"
  LOG_LEVEL: "debug"
  DB_CONNECTION: "mysql"
  DB_HOST: "mysql"
  DB_PORT: "3306"
  DB_DATABASE: "laravel_in_kubernetes"
  BROADCAST_DRIVER: "log"
  CACHE_DRIVER: "file"
  FILESYSTEM_DRIVER: "local"
  QUEUE_CONNECTION: "sync"
  
  # Update the Session driver to Redis, based off part-2 of series
  SESSION_DRIVER: "redis"
  SESSION_LIFETIME: "120"
  MEMCACHED_HOST: "memcached"
  REDIS_HOST: "redis"
  REDIS_PORT: "6379"
  MAIL_MAILER: "smtp"
  MAIL_HOST: "mailhog"
  MAIL_PORT: "1025"
  MAIL_ENCRYPTION: "null"
  MAIL_FROM_ADDRESS: "null"
  MAIL_FROM_NAME: "${APP_NAME}"
  AWS_DEFAULT_REGION: "us-east-1"
  AWS_BUCKET: ""
  AWS_USE_PATH_STYLE_ENDPOINT: "false"
  PUSHER_APP_ID: ""
  PUSHER_APP_CLUSTER: "mt1"
  MIX_PUSHER_APP_KEY: "${PUSHER_APP_KEY}"
  ```
  ```bash
  apiVersion: v1
kind: Secret
metadata:
  name: laravel-in-kubernetes
type: Opaque
stringData:
  APP_KEY: "base64:eQrCXchv9wpGiOqRFaeIGPnqklzvU+A6CZYSMosh1to="
  DB_USERNAME: "sail"
  DB_PASSWORD: "password"
  REDIS_PASSWORD: "null"
  MAIL_USERNAME: "null"
  MAIL_PASSWORD: "null"
  AWS_ACCESS_KEY_ID: ""
  AWS_SECRET_ACCESS_KEY: ""
  PUSHER_APP_KEY: ""
  PUSHER_APP_SECRET: ""
  MIX_PUSHER_APP_KEY: "${PUSHER_APP_KEY}"
  ```
  上面两个分别是config 和 secrect，用 secret 的目的就是为了信息安全。
  在我们真实使用中，要把上面的参数替换成自己的，尤其是 `APP_ENV: "production"` 和 `APP_DEBUG: "false"` 一定要改成这样。

这两个配置文件都是保存在 `common` 目录下的 `app-config.yaml` 和 `app-secret.yaml` 文件中。当我们编辑完文件后，执行下面的命令
```bash
kubectl apply -f common/
```
这样就把整个目录的文件都应用了，每次修改完都可以应用整个目录，因为只有被修改的会触发更新，没有修改的会跳过。

注意啊，当我们真实部署以后，更新 config or secret 后，并不会触发 pods 的滚动更新。所以，我们采用 [https://jimmysong.io/kubernetes-handbook/concepts/configmap-hot-update.html](https://jimmysong.io/kubernetes-handbook/concepts/configmap-hot-update.html) 这篇文章的 `ConfigMap 更新后滚动更新 Pod` 这个段落的方案，给 pod 增加一个 注解，每次更新后，更新注解的方式来更新pod ，当然这是我们管理少量镜像的方案，要是项目大了，还得搞配置中心才行。
![](https://www.cimple.ink/static/images/202303091153736.png)
注解的方式如上图。

### fpm 部署
创建 `fpm` 目录，在此目录下创建 `deployment.yml` 文件
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: laravel-in-kubernetes-fpm
  labels:
    tier: backend
    layer: fpm
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: backend
      layer: fpm
  template:
    metadata:
      labels:
        tier: backend
        layer: fpm
      annotations:
        version/config: "xxxx"
    spec:
      initContainers:
        - name: migrations
          image: [your_registry_url]/cli:v0.0.1
          command:
            - php
          args:
            - artisan
            - migrate
            - --force
          envFrom:
            - configMapRef:
                name: laravel-in-kubernetes
            - secretRef:
                name: laravel-in-kubernetes
      containers:
        - name: fpm
          image: [your_registry_url]/fpm_server:v0.0.1
          ports:
            - containerPort: 9000
          envFrom:
            - configMapRef:
                name: laravel-in-kubernetes
            - secretRef:
                name: laravel-in-kubernetes
```
这里注意点就很少了，修改配置版本号，以及自己的镜像位置。可以看到我们通过 `envFrom` 的方式因为如了我们上面的配置，name 就是在 config 和 secret 里面设置的 meta。还有一个就是 `initContainers` ，我们通过这个方式来初始化我们的数据库。
在`fpm` 目录下继续创建 `service.yml` 文件
```bash
apiVersion: v1
kind: Service
metadata:
  name: laravel-in-kubernetes-fpm
spec:
  selector:
    tier: backend
    layer: fpm
  ports:
    - protocol: TCP
      port: 9000
      targetPort: 9000
```
### web 部署
创建 `web` 目录，且创建 `deploy.yml` 和 `secret.yml` 文件
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: laravel-in-kubernetes-webserver
  labels:
    tier: backend
    layer: webserver
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: backend
      layer: webserver
  template:
    metadata:
      labels:
        tier: backend
        layer: webserver
    spec:
      containers:
        - name: webserver
          image: [your_registry_url]/web_server:v0.0.1
          ports:
            - containerPort: 80
          env:
            # Inject the FPM Host as we did with Docker Compose
            - name: FPM_HOST
              value: laravel-in-kubernetes-fpm:9000
```
```bash
apiVersion: v1
kind: Service
metadata:
  name: laravel-in-kubernetes-webserver
spec:
  selector:
    tier: backend
    layer: webserver
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

这里也没什么的注意的了，只有 deploy 里面的通过 `env` 的方式，设置 `FPM_HOST`。这里面的网站，对应上面 `fpm service` 的 `meta name`。

### 队列部署
创建 `queue-works` 目录。 编写 `deployment-default.yml` 文件。
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: laravel-in-kubernetes-queue-worker-default
  labels:
    tier: backend
    layer: queue-worker
    queue: default
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: backend
      layer: queue-worker
      queue: default
  template:
    metadata:
      labels:
        tier: backend
        layer: queue-worker
        queue: default
    spec:
      containers:
        - name: queue-worker
          image: [your_registry_url]/cli:v0.0.1
          command:
            - php
          args:
            - artisan
            - queue:work
            - --queue=default
            - --max-jobs=200
          ports:
            - containerPort: 9000
          envFrom:
            - configMapRef:
                name: laravel-in-kubernetes
            - secretRef:
                name: laravel-in-kubernetes
```

这里就采用我们之前编译的 cli 来执行队列了。没什么注意的，就是看 laravel 官方文档基本都能明白了。如果有多个队列怎么办呢？在创建 `deployment-{queueName}.yml` 文件即可，再把配置里面的 `--queue={queueName}` 替换为具体的队列名称就好了。

日后如果想更新数据库，我们就可以通过队列来预埋一个命令，就可以实时更新数据库了。

### 定时任务
创建 `cron` 目录，创建 `cronjob.yml` 文件。
```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: laravel-in-kubernetes-scheduler
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: scheduler
            image: [your_registry_url]/cli:v0.0.1
            command:
              - php
            args:
              - artisan
              - schedule:run
            envFrom:
              - configMapRef:
                  name: laravel-in-kubernetes
              - secretRef:
                  name: laravel-in-kubernetes
          restartPolicy: OnFailure
```
文中还说了，用之前编译的cron 在写一个deployment。但是我觉得还是用 cronJob 比较合适。所以那个方案我就没记录。

## 总计
至此，所有的笔记就全都记录好了，还有两个弄 lb 和 证书的，我就没弄，因为依赖平台，如果是云平台就用云平台的就好，如果自建还是看上面那两篇文章。
好了，又弄完一个大活，接下来会休息休息，看看书，然后几句我的那个小组件。计划了。

---

> 作者:   
> URL: http://localhost:1313/posts/personal-summary-of-laravel-in-kubernetes-series-of-articles-2/  

