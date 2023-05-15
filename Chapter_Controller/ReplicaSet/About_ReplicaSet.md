### 레플리케이션 컨트롤러

: 쿠버네티스에서 지정한 숫자만큼 파드의 수가 유지되도록 조절하는 컨트롤러. 파드에 장애가 생기거나 이상이 있을 때, 수동으로 하는 것이 아닌 컨트롤러를 통해 유지하면 훨씬 쉽다. 요즘은 디플로이먼트를 사용한다.

특징:

- 등호 기반 (레이블 일치 여부)이 아닌 “집합 기반 셀렉터”를 지원한다. (in, notin exists)
- rolling-update (순차적 업데이트) 지원 X

- 레플리카세트 (ReplicaSet)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  template:
    metadata:
      name: nginx-replicaset
      labels:
        app: nginx-replicaset
    spec:
      containers:
      - name: nginx-replicaset
        image: nginx
        ports:
      - containerPort: 80
  replicas: 3 #유지할 파드의 수도
  selector: #관리할 파드의레이블, 하위 설정을 기준으로 레이블을 판단한다.
     matchLabels:
      app: nginx-replicaset
```

- Desired state를 나타내는 template를 통해 목표 상태를 설정한다.
- 레이블은 nginx-replicaset으로 설정한다.
- 컨테이너는 파드당 1개씩 총 세 개 (replicas : 3)를 유지하도록 한다.
- selector를 통해 해당 레플리카 세트가 어떤 레이블이 담긴 파드를 관리할지 지정해준다.
⇒ 레이블이 다르면 관리하는 파드가 아니라고 판단하여 replicas 갯수에 반영하지 않는다.

**레이블로 관리하는 파드**

```bash
> kc get pods
NAME                     READY   STATUS              RESTARTS   AGE
nginx-replicaset-5v97g   1/1     Running             0          7s
nginx-replicaset-8mpsn   0/1     ContainerCreating   0          7s
nginx-replicaset-tcnjd   1/1     Running             0          7s
```

```bash
kubectl edit pod nginx-replicaset-tcnjd #파드의 레이블을 수정해본다.
```

⇒ 여기서 .metadata.labels.app 을 다른 레이블로 수정하면 해당 파드는 레플리카 세트에서 인식을 하지 못한다. 따라서, 1개 파드가 사라졌다고 인식되어 새로운 파드를 형성한다.
+ kc는 kubectl 명령어를 alias를 통해 매핑한 명령어다. (구글에 치면 해당 방법 나옴)

```bash
kc get pods
NAME                     READY   STATUS              RESTARTS   AGE
nginx-replicaset-5v97g   1/1     Running             0          72s
nginx-replicaset-8mpsn   1/1     Running             0          72s
nginx-replicaset-kznbb   0/1     ContainerCreating   0          2s
nginx-replicaset-tcnjd   1/1     Running             0          72s>
```

⇒ 다음과 같이 nginx-replicaset-kznbb 파드가 새로 생성되었음을 알 수 있다.
