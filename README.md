# Домашнее задание к занятию "Обновление приложений" dev-17_kuber-homeworks-3.4-yakovlev_vs
"Обновление приложений"

### Цель задания

Выбрать и настроить стратегию обновления приложения.

### Чеклист готовности к домашнему заданию

1. Кластер k8s.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Документация Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)
2. [Статья про стратегии обновлений](https://habr.com/ru/companies/flant/articles/471620/)

-----

### Задание 1. Выбрать стратегию обновления приложения и описать ваш выбор.

1. Имеется приложение, состоящее из нескольких реплик, которое требуется обновить.
2. Ресурсы, выделенные для приложения ограничены, и нет возможности их увеличить.
3. Запас по ресурсам в менее загруженный момент времени составляет 20%.
4. Обновление мажорное, новые версии приложения не умеют работать со старыми.
5. Какую стратегию обновления выберете и почему?

#### Решение

Вариант 1. Если приложение уже протестировано ранее, то лучшим вариантом наверное будет использовать стратегию обновления Rolling Update, 
указанием параметров `maxSurge` `maxUnavailable` для избежания ситуации с нехваткой ресурсов. Проводить обновление следует естественно в менее загруженный момент времени сервиса.
При данной стратегии(Rolling Update) k8s постепенно заменит все поды без ущерба производительности. И если что-то пойдет не так, можно будет быстро откатится к предыдущему состоянию.

Вариант 2. Можно использовать Canary Strategy. Также указав параметры `maxSurge` `maxUnavailable` чтобы избежать нехватки ресурсов. 
Это позволит нам протестировать новую версию программы на реальной пользовательской базе(группа может выделяться по определенному признаку) без обязательства полного развертывания. 
После тестирования и собирания метрик пользователей можно постепенно переводить поды к новой версии приложения.


### Задание 2. Обновить приложение.

1. Создать deployment приложения с контейнерами nginx и multitool. Версию nginx взять 1.19. Кол-во реплик - 5.
2. Обновить версию nginx в приложении до версии 1.20, сократив время обновления до минимума. Приложение должно быть доступно.
3. Попытаться обновить nginx до версии 1.28, приложение должно оставаться доступным.
4. Откатиться после неудачного обновления.

#### Решение

1. Создать deployment приложения с контейнерами nginx и multitool. Версию nginx взять 1.19. Кол-во реплик - 5.

- [deployments.yaml](file/deployments.yaml)
- [svc.yaml](file/svc.yaml)

```bash
$ kubectl apply -f file/deployments.yaml 
deployment.apps/netology-deployment created

$ kubectl apply -f file/svc.yaml 
service/mysvc created

$ kubectl get pod
NAME                                   READY   STATUS    RESTARTS   AGE
netology-deployment-66d67f955b-7cghv   2/2     Running   0          38m
netology-deployment-66d67f955b-wd76r   2/2     Running   0          38m
netology-deployment-66d67f955b-f5njq   2/2     Running   0          38m
netology-deployment-66d67f955b-fxfkp   2/2     Running   0          38m
netology-deployment-66d67f955b-f9tfq   2/2     Running   0          38m

$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP             79d
mysvc        ClusterIP   10.152.183.130   <none>        9001/TCP,9002/TCP   11m
```

```bash
$ kubectl get pod netology-deployment-66d67f955b-fxfkp -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: 91f482b1907e11b8fa96307ac22a04f90dbe95e8f1aa4bd94691754c5dd3a03e
    cni.projectcalico.org/podIP: 10.1.128.235/32
    cni.projectcalico.org/podIPs: 10.1.128.235/32
  creationTimestamp: "2023-05-01T17:31:03Z"
  generateName: netology-deployment-66d67f955b-
  labels:
    app: main
    pod-template-hash: 66d67f955b
  name: netology-deployment-66d67f955b-fxfkp
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: netology-deployment-66d67f955b
    uid: 1abfebe4-4f9e-4d4d-b3c4-dd4eb7222d24
  resourceVersion: "454900"
  uid: c3e8db23-62ea-401f-a728-fc24871fdae3
spec:
  containers:
  - image: nginx:1.19
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-t87z8
      readOnly: true
  - env:
    - name: HTTP_PORT
      value: "8080"
    - name: HTTPS_PORT
      value: "11443"
    image: wbitt/network-multitool
    imagePullPolicy: Always
    name: network-multitool
    ports:
    - containerPort: 8080
      name: http-port
      protocol: TCP
    - containerPort: 11443
      name: https-port
      protocol: TCP
    resources:
      limits:
        cpu: 10m
        memory: 20Mi
      requests:
        cpu: 1m
        memory: 20Mi
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-t87z8
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: microk8s
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-t87z8
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2023-05-01T17:31:04Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2023-05-01T17:31:25Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2023-05-01T17:31:25Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2023-05-01T17:31:04Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://3155a855776df499063ae2fad90e263a54923351e0c4823289526aac344ac6d0
    image: docker.io/wbitt/network-multitool:latest
    imageID: docker.io/wbitt/network-multitool@sha256:82a5ea955024390d6b438ce22ccc75c98b481bf00e57c13e9a9cc1458eb92652
    lastState: {}
    name: network-multitool
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2023-05-01T17:31:24Z"
  - containerID: containerd://645d031a24761b2f7e1c9473546672e58beb187311db5f570bb09c48008f41b8
    image: docker.io/library/nginx:1.19
    imageID: docker.io/library/nginx@sha256:df13abe416e37eb3db4722840dd479b00ba193ac6606e7902331dcea50f4f1f2
    lastState: {}
    name: nginx
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2023-05-01T17:31:19Z"
  hostIP: 192.168.1.88
  phase: Running
  podIP: 10.1.128.235
  podIPs:
  - ip: 10.1.128.235
  qosClass: Burstable
  startTime: "2023-05-01T17:31:04Z"
```

2. Обновить версию nginx в приложении до версии 1.20, сократив время обновления до минимума. Приложение должно быть доступно.

Обновляем. Меняем в `deployment.yaml` параметр image: nginx:1.19 на 1.20. 

Выбираем и добавляем параметры стратегии обновления для того чтобы приложение было всегда доступно.

```yaml
strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

```bash
$ kubectl exec deployment/netology-deployment -- curl mysvc:9002
Defaulted container "nginx" out of: nginx, network-multitool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   157  100   157    0     0   7476      0 --:--:-- --:--:-- --:--:--  7476
WBITT Network MultiTool (with NGINX) - netology-deployment-66d67f955b-wd76r - 10.1.128.239 - HTTP: 8080 , HTTPS: 11443 . (Formerly praqma/network-multitool)

$ kubectl apply -f file/deployments.yaml 
deployment.apps/netology-deployment configured

$ kubectl get pod -o wide
NAME                                   READY   STATUS              RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
netology-deployment-66d67f955b-7cghv   2/2     Running             0          52m   10.1.128.246   microk8s   <none>           <none>
netology-deployment-66d67f955b-fxfkp   2/2     Running             0          52m   10.1.128.235   microk8s   <none>           <none>
netology-deployment-7fddc586d8-vxplv   2/2     Running             0          51s   10.1.128.234   microk8s   <none>           <none>
netology-deployment-66d67f955b-wd76r   2/2     Terminating         0          52m   10.1.128.239   microk8s   <none>           <none>
netology-deployment-7fddc586d8-g8x2x   2/2     Running             0          53s   10.1.128.200   microk8s   <none>           <none>
netology-deployment-7fddc586d8-sd2z2   0/2     ContainerCreating   0          18s   <none>         microk8s   <none>           <none>
netology-deployment-66d67f955b-f5njq   2/2     Terminating         0          52m   10.1.128.238   microk8s   <none>           <none>

$ kubectl exec deployment/netology-deployment -- curl mysvc:9002
Defaulted container "nginx" out of: nginx, network-multitool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0WBITT Network MultiTool (with NGINX) - netology-deployment-7fddc586d8-vxplv - 10.1.128.234 - HTTP: 8080 , HTTPS: 11443 . (Formerly praqma/network-multitool)
100   157  100   157    0     0   1286      0 --:--:-- --:--:-- --:--:--  1276
```


```bash
$ kubectl get pod -o wide
NAME                                   READY   STATUS    RESTARTS   AGE     IP             NODE       NOMINATED NODE   READINESS GATES
netology-deployment-7fddc586d8-vxplv   2/2     Running   0          3m2s    10.1.128.234   microk8s   <none>           <none>
netology-deployment-7fddc586d8-g8x2x   2/2     Running   0          3m4s    10.1.128.200   microk8s   <none>           <none>
netology-deployment-7fddc586d8-sd2z2   2/2     Running   0          2m29s   10.1.128.245   microk8s   <none>           <none>
netology-deployment-7fddc586d8-pxwkz   2/2     Running   0          2m11s   10.1.128.208   microk8s   <none>           <none>
netology-deployment-7fddc586d8-h7j5d   2/2     Running   0          2m2s    10.1.128.213   microk8s   <none>           <none>
```

Все поды постепенно обновились, при этом приложение оставалось доступным через сервис `mysvc`.

```bash
$ kubectl describe deployment netology-deployment
Name:                   netology-deployment
Namespace:              default
CreationTimestamp:      Mon, 01 May 2023 20:31:03 +0300
Labels:                 app=main
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=main
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  app=main
  Containers:
   nginx:
    Image:        nginx:1.20
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
   network-multitool:
    Image:       wbitt/network-multitool
    Ports:       8080/TCP, 11443/TCP
    Host Ports:  0/TCP, 0/TCP
    Limits:
      cpu:     10m
      memory:  20Mi
      HTTPS_PORT:  11443
    Mounts:        <none>
  Volumes:         <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   netology-deployment-7fddc586d8 (5/5 replicas created)
Events:          <none>

Valdem88@Valdem-PCnew MINGW64 ~/PycharmProjects/dev-17_kuber-homeworks-3.4-yakovlev_vs (main)
$ kubectl get svc
Error: unknown command "пÑget" for "kubectl"
Run 'kubectl --help' for usage.
```

3. Попытаться обновить nginx до версии 1.28, приложение должно оставаться доступным.

Меняем в `deployment.yaml` параметр image: nginx:1.19 на 1.28.

```bash
$ kubectl apply -f file/deployments.yaml 
deployment.apps/netology-deployment configured
```

Получаем

```bash
$ kubectl get pod
NAME                                   READY   STATUS             RESTARTS   AGE
netology-deployment-7fddc586d8-g8x2x   2/2     Running            0          14m
netology-deployment-7fddc586d8-sd2z2   2/2     Running            0          14m
netology-deployment-7fddc586d8-pxwkz   2/2     Running            0          13m
netology-deployment-7fddc586d8-h7j5d   2/2     Running            0          13m
netology-deployment-6c8c49d66d-j4jcz   1/2     ImagePullBackOff   0          89s
netology-deployment-6c8c49d66d-6rlnh   1/2     ImagePullBackOff   0          89s
```

при этом

```bash
$ kubectl exec deployment/netology-deployment -- curl mysvc:9002
Defaulted container "nginx" out of: nginx, network-multitool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   157  100   157    0     0   5607      0 --:--:-- --:--:-- --:--:--  6280
WBITT Network MultiTool (with NGINX) - netology-deployment-7fddc586d8-sd2z2 - 10.1.128.245 - HTTP: 8080 , HTTPS: 11443 . (Formerly praqma/network-multitool)
```

4. Откатиться после неудачного обновления.

```bash
$ kubectl rollout status deployment netology-deployment
Waiting for deployment "netology-deployment" rollout to finish: 2 out of 5 new replicas have been updated...
```

```bash
$ kubectl rollout undo deployment netology-deployment
deployment.apps/netology-deployment rolled back

$ kubectl get pod
NAME                                   READY   STATUS    RESTARTS   AGE
netology-deployment-7fddc586d8-g8x2x   2/2     Running   0          21m
netology-deployment-7fddc586d8-sd2z2   2/2     Running   0          21m
netology-deployment-7fddc586d8-pxwkz   2/2     Running   0          21m
netology-deployment-7fddc586d8-h7j5d   2/2     Running   0          20m
netology-deployment-7fddc586d8-dmhfq   2/2     Running   0          14s
```

## Дополнительные задания (со звездочкой*)

**Настоятельно рекомендуем выполнять все задания под звёздочкой.**   Их выполнение поможет глубже разобраться в материале.   
Задания под звёздочкой дополнительные (необязательные к выполнению) и никак не повлияют на получение вами зачета по этому домашнему заданию. 

### Задание 3*. Создать Canary deployment.

1. Создать 2 deployment'а приложения nginx.
2. При помощи разных ConfigMap сделать 2 версии приложения (веб-страницы).
3. С помощью ingress создать канареечный деплоймент, чтобы можно было часть трафика перебросить на разные версии приложения.

### Правила приема работы

1. Домашняя работа оформляется в своем Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md