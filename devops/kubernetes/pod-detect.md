# K8S Pod状态检测学习

K8S 作为一个容器编排服务，他有一个显著的功能就是服务监控和自愈。那这个功能依靠其内部的什么实现的呢？今天就来学习其中一个：Pod 的存活、就绪和启动探测器。

- 启动探测器：用来判断应用程序容器什么时候启动了；
- 就绪探测器：用来判断什么时候容器已准备好接收流量；
- 存活探测器：用来判断什么时候需要重启容器；

## 存活探测器

存活探测器使用`livenessProbe`关键字设置。配置方式支持如下：

- 存活命令
- HTTP请求接口
- TCP端口
- 命名端口

### 存活命令

基于下面配置，我们创建一个基于`busybox`镜像的容器，配置如下：

```yam
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

我们看`livenessProbe`下的配置项

- `exec`: 执行命令式存活性探测，检测命令为：`cat /tmp/healthy`;
- `initialDelaySeconds`: 执行第一次探测前应该等待的时间。这里为5s;
- `periodSeconds`：间隔时间。每隔5s执行一次检测；

在 `30s` 内查看 Pod 事件：

```shel
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  3s    default-scheduler  Successfully assigned default/liveness-exec to minikube
  Normal  Pulling    2s    kubelet            Pulling image "busybox"
```

`35s`后，再次查看时间，可以看到容器已被杀死：

```shell
Events:
  Type     Reason     Age   From               Message
  ----     ------     ----  ----               -------
  Normal   Scheduled  35s   default-scheduler  Successfully assigned default/liveness-exec to minikube
  Normal   Pulling    34s   kubelet            Pulling image "busybox"
  Normal   Pulled     31s   kubelet            Successfully pulled image "busybox" in 3.641460146s
  Normal   Created    31s   kubelet            Created container liveness
  Normal   Started    31s   kubelet            Started container liveness
  Warning  Unhealthy  0s    kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```

在等`30s`，可以看到容器重启了：

```shell
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  76s                default-scheduler  Successfully assigned default/liveness-exec to minikube
  Normal   Pulled     72s                kubelet            Successfully pulled image "busybox" in 3.641460146s
  Normal   Created    72s                kubelet            Created container liveness
  Normal   Started    72s                kubelet            Started container liveness
  Warning  Unhealthy  31s (x3 over 41s)  kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    31s                kubelet            Container liveness failed liveness probe, will be restarted
  Normal   Pulling    1s (x2 over 75s)   kubelet            Pulling image "busybox"
```

### HTTP 请求接口

`HTTP`存活一般使用容器内服务提供的心跳接口，下面是配置文件：

```yam
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

看配置文件中`livenessProbe`关键字下的配置参数：

- `httpGet`: 使用`HTTP GET` 方法，还可以配置路径、端口、头部参数等； 
- `initialDelaySeconds`: 执行第一次探测前等待的时间；
- `periodSeconds`: 检测间隔时间；

容器中的代码如下：当运行时间超过`10s`后，将返回500错误

```go
http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
    duration := time.Now().Sub(started)
    if duration.Seconds() > 10 {
        w.WriteHeader(500)
        w.Write([]byte(fmt.Sprintf("error: %v", duration.Seconds())))
    } else {
        w.WriteHeader(200)
        w.Write([]byte("ok"))
    }
})
```

### TCP 端口

也可以配置TCP 套接字端口，`kubelet`会尝试与指定端口的套接字建立连接，如果连接成功建立，则探测结果健康；否则，就认为有问题。

下面是 TCP 端口配置的样例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

看关键字`livenessProbe`的配置：

- `tcpSocket`: 指定使用TCP套接字类型，可以配置端口参数；
- `initialDelaySeconds`: 第一次探测时需要等待的时间；
- `periodSeconds`：探测执行间隔；

### 使用命名端口

即在 `HTTP` 和 `TCP` 方式中，使用容器中定义的端口名称。

## 启动探测器

`kubelet` 通过启动探测器可以知道应用程序容器什么时候启动了。这样，就可以控制容器在启动成功后再进行后续的就绪和存活性检测。确保就绪和存活性的检测不会影响到容器的正常启动。对于一些启动时间过长的容器，可以避免在启动过程就因存活性检测而被杀掉。

启动探测时使用 `startupProbe` 关键字配置：

```yam
startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```

上面配置中，启动探测器将会有 300s 来完成启动(failureThreshold * periodSeconds )；一旦有一次启动探测成功，则存活性探测任务会接管对容器的探测，对容器死锁做出响应。如果启动探测一直没有成功，容器会在300s后被杀死，依据 `restartPolicy` 来设置 Pod状态。

## 就绪探测器

有时候，应用程序启动后，还不能立即提供服务，需要做一些额外的初始化或加载数据的工作。这种情况下，既不想杀死容器，也不想负责均衡器发送流量到容器中。K8S中通过就绪探测器，来确定容器的就绪状态。

就绪探测器使用`readinessProbe`关键字进行配置：（配置参数与存活性探测相似）

```yaml
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

> 注意：
>
> 1. 就绪探测器在容器的整个生命周期中保持运行状态；
>
> 2. 存活性探测器不等待就绪性探测器成功。

