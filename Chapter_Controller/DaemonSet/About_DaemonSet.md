## 데몬세트 (Daemon)

: 클러스터 내의 전체 노드에 항상 실행되어야 하는 파드를 실행할 때 사용하는 컨트롤러 (ex. 로그 수집 파드 등 모든 노드에 항상 존재해야 하는 파드를 실행할 때 사용하는 컨트롤러)

![img](https://github.com/synoti21/kubernetes-exercise/assets/58936172/e364178b-4942-42d1-b2a3-2165b2f922a6)

⇒ 모든 노드에 로그 수집 파드가 있어야 하므로 로그 수집용 데몬세트를 실행하면 자동으로 전체 노드에 해당 파드를 실행하게 된다.

데몬 세트 예시 yaml

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-elasticsearch
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-elasticsearch
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      containters:
      - name: fluentd-elasticsearch
        image: fluent/flunetd-kubernetes-daemonset:elasticsearch
        env:
          - name: testenv
            value: value
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
```

- 보통 로그 수집기의 경우 k8s 관리용 파드나 설정에 해당하므로 kube-system에 해당하므로, 별도로 네임스페이스를 지정해준다.
- 업데이트 방법은 RollingUpdate (순차적 업데이트), 또는 OnDelete (파드를 직접 지워야 새로운 템플릿 버전의 파드가 실행됨) 가 있다.

```yaml
kc get daemonset -n kube-system
NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
fluentd-elasticsearch   1         1         0       1            0           <none>                   31m
kube-proxy              1         1         1       1            1           kubernetes.io/os=linux   48d
```

⇒ namespace가 kube-system으로 지정되었으므로 해당 데몬 세트가 뜨는 것을 확인할 수 있다.
