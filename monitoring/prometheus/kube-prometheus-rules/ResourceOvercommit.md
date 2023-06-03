## KubeResourceOverCommit alert ([CPU](https://runbooks.prometheus-operator.dev/runbooks/kubernetes/kubecpuovercommit/#kubecpuovercommit)/[Memory](https://runbooks.prometheus-operator.dev/runbooks/kubernetes/kubememoryovercommit/))

정의 : 클러스터의 내의 임의의 노드가 죽었을 때 기존의 살아있는 노드만을 가지고는 모든 팟을 schedule할 수 없는 상태

    ```bash
    sum(namespace_memory:kube_pod_container_resource_requests:sum) - (sum(kube_node_status_allocatable{resource="memory"}) - max(kube_node_status_allocatable{resource="memory"})) > 0 and (sum(kube_node_status_allocatable{resource="memory"}) - max(kube_node_status_allocatable{resource="memory"})) > 0
    ```
    
    즉 노드들의 가용 자원의 총합 > 가장 가용 자원이 많은 노드의 가용 자원 (== 가용자원이 있는 노드의 수가 2개 이상?)이면서
    클러스터 내의 모든 컨테이너 자원 requests의 총합 > 가장 가용자원이 많은 노드를 제외한 노드들의 가용자원 총합 ( == 가용자원이 가장 많은 노드가 죽은 최악의 상황에서 모든 컨테이너의 자원 requests를 소화할 수 없다)일 경우
    

실제로 해당 alert가 발생했던 서버 node의 상태를 확인해봤을 때, 굉장히 많은 자원의 양을 할당해놓았음을 확인할 수 있었다.
즉, 정의된대로 임의의 노드가 죽었을 때 살아있는 노드만을 가지고는 모든 팟을 schedule할 수 없는 상태가 될 수도 있다. 

여기서 들었던 의문.

애초에 limit이 저렇게 넘치게 팟을 스케줄링해야하는 상황이면, 애초에 위험부담을 하면서까지 팟을 할당하면 안될 것 같으므로 aws에서 관리해주는 autoscaler가 scale out해줘야 하는게 아닌가?

⇒ 아니다! 일단 overcommit된 상태는 팟을 할당하면 안되는 상태가 아니다. 팟을 스케줄링 할 때 스케줄러는 필터링 과정에서 자원 requests만 바라보고 있다. 즉, requests만 커버가 가능하다면 필터링되는 대상이 아니다 (스코어링 할 때는 고려하는 하나의 지표이긴 하다)

따라서 충분히 팟을 스케줄할 수 있는 상태이기 때문에 scale out되지 않는다

### 어떻게 하면 alert가 안올까?

여러 방법이 있다.

1. 중요하지 않은 alert라고 판단됐다면 ignore
    1. 노드가 죽지 않는 이상 웬만하면 위험하지는 않은 상태
2. 노드별로 과도한 자원을 사용하지 않도록 조절한다.
   1. 노드 수를 임의로 늘려서 하나의 노드가 죽더라도 모든 팟을 스케줄링 할 수 있도록 한다.
      1. aws에서 관리해주는 autoscaler를 사용중이라면, 사실상 사용하기 어려운 선택지.
   2. 노드의 오토스케일링 옵션을 조절하여 일정 자원 이상을 사용하면 스케일 아웃되도록 설정된다.