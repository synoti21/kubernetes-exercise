## 스테이트풀세트

: 상태가 있는 파드들을 관리할 때 사용하는 컨트롤러
![img-2](https://github.com/synoti21/kubernetes-exercise/assets/58936172/c02fadc0-745b-4240-9a1b-5a8292c1465d)

⇒ 순서나 있어야 할 데이터를 지정하는 등, “상태”가 존재해야 한다.

스테이트풀세트 예시

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-statefulsets-service
  labels:
    app: nginx-statefulsets-service
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx-statefulsets-service
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx-statefulset
  serviceName: "nginx-statefulsets-service"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx-statefulset
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx-statefulset
        image: nginx
        ports:
        - containerPort: 80
          name: web
```

- 서비스의 메타데이터의 이름을 지정하고, 스테이트풀 세트에서 만들어진 파드의 이름을 조합하면 도메인을 만들 수 있다. (파드이름.서비스이름)
- terminationGracePeriodSeconds는 바로 종료가 아닌, 테스크들을 모두 마무리하고 정상 종료를 하도록 대기시간을 설정한다.

kubectl apply -f statefulset.yaml을 실행한 후 상태를 확인하면 다음과 같다.

```bash
> kc get svc,statefulset,pods
NAME                                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes                   ClusterIP   10.96.0.1    <none>        443/TCP   47d
service/nginx-statefulsets-service   ClusterIP   None         <none>        80/TCP    3m32s

NAME                   READY   AGE
statefulset.apps/web   3/3     3m31s

NAME        READY   STATUS    RESTARTS   AGE
pod/web-0   1/1     Running   0          2m43s
pod/web-1   1/1     Running   0          2m29s
pod/web-2   1/1     Running   0          2m26s
```

- 이전의 컨트롤러와 다르게, 스테이트풀 세트는 파드들을 순서대로 관리한다.
- 숫자가 작은 순서대로 실행이 되며, 앞선 숫자의 파드가 실행이 되어야만 뒤의 파드들이 실행이 된다.
- 삭제는 거꾸로 큰 순서대로 삭제가 된다.
- podManagementPolicy가 OrderedReady일 경우, 위의 경우처럼 순서대로 관리하고, Parallel일 경우 순서와 상관없이 관리한다.
