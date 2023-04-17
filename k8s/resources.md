# Pod Resources

일반적으로 일컫는 Pod의 자원은 CPU 및 메모리가 있다 (이외 hugepages도 있다)

Pod에서 컨테이너에 대한 자원 요청(request) 및 제한(limit)을 지정할 수 있으며, 요청은 자원의 lowerBound 역할을, limit은 upperBound 역할을 한다. 

만약, 어떤 컨테이너가 제한된 자원량을 넘어서 사용할 경우, cpu의 경우 throttling이 걸리고 메모리의 경우 컨테이너가 OOM kill되며 재시작한다.

kube-scheduler는 요청 정보를 가지고 Pod이 배치될 노드를 결정한다. 즉 Pod을 생성할 때, 스케줄러는 각 노드가 Pod에 제공할 수 있는 각 리소스의 최대 가용치를 확인하여 스케줄링한다. 주의할 점은, 노드의 리소스 최대 가용치를 확인하는데에 실패할 경우에는 Pod을 스케줄링하지 않는다.

kubelet은 컨테이너 런타임에 해당 컨테이너의 자원 요청/제한을 전달하여 이를 실행(?)하는 것을 도운다. 특히 자원 사용량을 제한하기 위해 컨테이너 런타임애서 적용된 커널 cgroup을 설정한다.
- CPU 제한은 강한 상한을 정의한다. 단위시간마다 해당 제한을 넘겼는지 확인하고 만약 초과하였다면 throttling된다.
- 메모리 제한은 해당 cgroup에 대한 메모리 사용량 상한을 정의한다. 만약 더 많은 메모리를 할당받으려 하면, OOM 서브시스템이 활성화되고 할당받으려고 하던 프로세스 중 하나를 죽인다. (만약 해당 프로세스의 PID가 1이고, 컨테이너가 재시작 가능으로 정의되어 있다면 k8s가 자동으로 재시작한다.)
- 메모리 제한은 메모리 기반 볼륨 (ex. emptyDir)의 페이지에도 적용될 수 있다.

만약 한 컨테이너가 메모리 요청을 초과했고, 이로 인해 스케줄링된 노드의 메모리가 부족해지면 컨테이너가 속한 Pod이 축출될 수 있다.

## 리소스 모니터링
kubelet은 Pod의 리소스 사용량을 Pod `status`에 포함하여 보고한다.

## 리소스 단위
### CPU
기본적으로 1CPU = 1000m CPU 단위를 사용한다. 1m 보다 작은 단위를 지정할 수 없다.
### 메모리
기본적으로 바이트 단위를 사용할 수 있다. 대부분 2의 거듭제곱 단위(Mi, Gi 등)를 사용한다.




TODO:
## CPUOverCommit