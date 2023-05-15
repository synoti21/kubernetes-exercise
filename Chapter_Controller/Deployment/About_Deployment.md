## 디플로이먼트 (Deployment)

: stateless 상태 앱을 배포할 때 사용하는 가장 기본적인 컨트롤러다. (웹 서비스와 같은 무중단 배포 등)

- 레플리카 세트를 관리하면서 좀 더 세밀하게 앱 배포를 관리함
- rolling update, 또는 롤백 기능, 잠시 중단 후 재배포 등 여러가지 기능을 세분화 한다.

예시 yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containters:
      - name: nginx-deployment
      image: nginx
      ports:
      - containerPort: 80
```

- ReplicaSet와 Deployment가 완전 분리된 컨트롤러가 아니다.
- Deployment는 파드의 수를 관리하는 ReplicaSet를 생성하며, 이 레플리카 세트를 관리하는 역할을 한다.
- 즉, 관리를 더 세부적으로 한다는 뜻

```bash
> kc get deploy,rs,rc,pods
```

디플로이먼트와 레플리카세트, 파드를 보면 다음과 같이 나오는 것을 알 수 있다.

```bash
> kc get deploy,rs,rc,pods
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           51s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-dfcbd5fd6   3         3         3       51s

NAME                                   READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-dfcbd5fd6-2qr8b   1/1     Running   0          51s
pod/nginx-deployment-dfcbd5fd6-4d2f5   1/1     Running   0          51s
pod/nginx-deployment-dfcbd5fd6-qhrc5   1/1     Running   0          51s
```

- 디플로이먼트와 그에 따른 레플리카세트, 그리고 세 개의 파드가 생성된 것을 알 수 있다.
- 디플로이먼트는 즉, 레플리카 세트를 통해 파드의 갯수를 유지하는 것을 알 수 있다.

### 디플로이먼트 업데이트

디플로이먼트를 변경하여 어떤 변경점이 일어나는지 알아보자.

```bash
> kc set image deployment/nginx-deployment nginx-deployment=nginx:1.9.1
```

```bash
> kc get deploy,rs,rc,pods
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     1            3           5m59s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7887b6dc45   1         1         0       5s
replicaset.apps/nginx-deployment-dfcbd5fd6    3         3         3       5m59s

NAME                                    READY   STATUS              RESTARTS   AGE
pod/nginx-deployment-7887b6dc45-7xbgl   0/1     ContainerCreating   0          5s
pod/nginx-deployment-dfcbd5fd6-2qr8b    1/1     Running             0          5m59s
pod/nginx-deployment-dfcbd5fd6-4d2f5    1/1     Running             0          5m59s
pod/nginx-deployment-dfcbd5fd6-qhrc5    1/1     Running             0          5m59s
```

- 변경 전 디플로이먼트인 replicaset.apps/nginx-deployment-dfcbd5fd6 대신, 새로운 디플로이먼트 replicaset.apps/nginx-deployment-7887b6dc45가 생성되었음을 알 수 있다.
- 또한, 변경점이 적용된 (image가 변경된) 새로운 파드가 하나둘씩 생성되고 있음을 알 수 있다.

변경이 완료되면 다음과 같이 된다.

```bash
> kc get deploy,rs,rc,pods
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           6m28s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7887b6dc45   3         3         3       34s
replicaset.apps/nginx-deployment-dfcbd5fd6    0         0         0       6m28s

NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7887b6dc45-7xbgl   1/1     Running   0          34s
pod/nginx-deployment-7887b6dc45-d5xrx   1/1     Running   0          17s
pod/nginx-deployment-7887b6dc45-nzxwp   1/1     Running   0          13s
```

- 변경 전 디플로이먼트의 DESIRED, CURRENT가 모두 0으로 되고, 새로운 파드들로 모두 대체되었음을 알 수 있다.
- 즉, 디플로이먼트의 정보가 변경될 때마다 새로운 레플리카세트가 생성되고, 그에 맞게 파드들도 변경된다.

### 디플로이먼트 롤백

앞서 디플로이먼트의 특징 중 하나는 세밀한 관리 기능이 있다고 했다. 그 중, 롤백에 관해서 알아보자.

```bash
> kc rollout history deploy nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

- 여러 개의 Revision이 나오는데, 이는 변경사항을 의미한다.

자세히 보려면 명령어에 revision 번호를 추가한다.

```bash
> kc rollout history deploy nginx-deployment --revision=2
deployment.apps/nginx-deployment with revision #2
Pod Template:
  Labels:	app=nginx-deployment
	pod-template-hash=7887b6dc45
  Containers:
   nginx-deployment:
    Image:	nginx:1.9.1
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

이전 버전으로 롤백 하기 위해서는 다음과 같은 명령어를 실행한다.

```bash
> kc rollout undo deploy nginx-deployment
```

```bash
> kc get deploy,rc,rs,pods
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           163m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7887b6dc45   0         0         0       157m
replicaset.apps/nginx-deployment-dfcbd5fd6    3         3         3       163m

NAME                                   READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-dfcbd5fd6-d2wvw   1/1     Running   0          8s
pod/nginx-deployment-dfcbd5fd6-rr57z   1/1     Running   0          5s
pod/nginx-deployment-dfcbd5fd6-wwzll   1/1     Running   0          12s
```

- 레플리카 세트를 보면 위의 버전은 신 버전, 밑의 버전은 구 버전인데, 구 버전으로 다시 레플리카세트가 운용되고 있음을 확인할 수 있다.
- 파드 역시 구 버전으로 변경되었음을 알 수 있다.
- 다만 리비전은 1이 아닌 새로운 3을 되었음을 알 수 있다.

리비전 숫자를 통해 되돌리려면 다음과 같은 명령어를 입력하면 된다.

```bash
> kc rollout undo deploy nginx-deployment --to-revision='리비전 숫자'
```

⇒ 마찬가지로 역시 리비전은 새로운 4가 된다.

```bash
> kc rollout history deploy nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
3         <none>
4         <none>
```

⇒ CHANGE-CAUSE에 내용을 기록하려면 annotation을 이용한다.

```yaml
...
labels:
  app: nginx-deployment
annotations:
  kubernetes.io/change-cause: version 1.10.1
```

파드 갯수를 조정하려면 scale 명령어를 사용하면 된다.

```bash
> kc scale deploy nginx-deployment --replicas=5
```

```bash
> kc get pods
NAME                               READY   STATUS              RESTARTS   AGE
nginx-deployment-dfcbd5fd6-fwlhl   1/1     Running             0          3m9s
nginx-deployment-dfcbd5fd6-hgh8w   1/1     Running             0          4s
nginx-deployment-dfcbd5fd6-m4cpb   1/1     Running             0          3m18s
nginx-deployment-dfcbd5fd6-qpz4l   1/1     Running             0          3m13s
nginx-deployment-dfcbd5fd6-z2mh9   0/1     ContainerCreating   0          4s
```

⇒ 다음과 같이 파드 갯수가 5개로 증가하였음을 확인할 수 있다. 

---
