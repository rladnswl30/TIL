# container
의존성이 포함된 표준화는 어디서든 동일한 동작을 한다.  
기본 host infra에서 Application을 분리하고, 다양한 클라우드/OS 환경에서 쉽게 배포

# container image
- Application을 실행하는 데 필요한 모든 것이 포함된 ready to run software package  
- 이미 실행 중인 컨테이너의 코드 변경 불가
- 변경하려면 새 이미지를 빌드하고 컨테이너 재생성해야함

### 이미지 이름
- 호스트 이름과 포트번호 포함 가능
  > ex) fictional.registry.example:10443/imagename
- tag를 추가하여 버전별로 식별 가능
  - 소문자/대문자/숫자/ _ / . / -  로 구성
  - 미지정 시 `latest` 의미
  - 롤백하기 어려울 수 있으므로 의미있는 tag 지정을 권장함

### 이미지 업데이트
#### image pull 정책
- default : 존재하는 이미지 외에 풀을 진행 `ifNotPresent`
- 필요 시, imagePullPolicy를 Always로 지정할 수 있음

### 다중 아키텍처 이미지
컨테이너 이미지 인덱스 제공  
컨테이너 아키텍처별로 여러 이미지를 제공하고 적합한 바이너리 이미지를 가져올 수 있음

### private registry
registry에서 이미지를 읽을 수 있는 키를 요구할 수 있도록 보안 구성.  
노드에서 도커를 실행하는 경우, 프라이빗 컨테이너 레지스트리를 인증하도록 도커 컨테이너 런타임을 구성할 수 있다.


# container runtime
- 컨테이너 실행을 담당하는 software
- docker, containerd, CRI-O, Kubernetes CRI를 구현한 모든 software

### Motivation
- 서로 다른 pod간에 runtime class를 설정하여 성능/보안 균형 유지  
- 때에 따라 pod마다 다른 runtime을 사용할 수 있음

### Setup
1. CRI (Container Runtime Interface)를 노드에 설정  
dockershim(docker), containerd, CRI-O

2. 상응하는 runtime class 리소스 생성   
  - `오브젝트 정의`
    ```yml
    apiVersion: node.k8s.io/v1  # 런타임클래스는 node.k8s.io API 그룹에 정의되어 있음
    kind: RuntimeClass
    metadata:
      name: myclass  # 런타임클래스는 해당 이름을 통해서 참조됨 # 유효한 DNS label 이름이어야 함
      # 런타임클래스는 네임스페이스가 없는 리소스임
    handler: myconfiguration  # 상응하는 CRI 설정의 이름임
    ```
  
  - pod에 `runtimeClassName 명시`
    ```yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mypod
    spec:
      runtimeClassName: myclass
    ```
    => runtimeClassName이 없으면 runtimeClass 기능이 비활성화된 것


# container 환경 변수
### container 환경
컨테이너에 다음과 같은 리소스 제공
- 하나의 이미지와 하나 이상의 볼륨이 결함된 file system
- 컨테이너 자신에 대한 정보
- 클러스터 내 다른 오브젝트에 대한 정보

### container 정보
- 컨테이너의 호스트 네임 = 컨테이너가 동작중인 Pod 이름
- hostname cmd, gethostname 함수 호출을 통해 가져올 수 있음

### cluster 정보
- 컨테이너가 생성될 때 실행중이던 모든 서비스 목록은 환경 변수로 해당 컨테이너에서 사용 가능
- 동일한 namespace 내에 있는 서비스 한정
  > ex) bar container에 매핑되는 foo service  
  > FOO_SERVICE_HOST=<서비스가 동작 중인 호스트>  
  > FOO_SERVICE_PORT=<서비스가 동작 중인 포트>  

# container life cycle hook
컨테이너가 life cycle동안의 이벤트에 의해 발동되는 코드를 실행하기 위해 life cycle hook framework를 사용하는 방법

### container hook
- PostStart : 컨테이너 생성 직후 실행
- PreStop : 컨테이너 종료 직전 호출. API 요청이나 liveness probe 실패, 선점, 자원 경합 등의 관리 이벤트로 발생함.

### hook handler
#### 구현
컨테이너는 hook handler를 구현하고 등록하여 해당 hook에 접근할 수 있음
- Exec : container의 cgroups, namespace안에서 pre-stop.sh cmd 실행
- HTTP : container 특정 end-point에 대해 http 요청 실행

#### 실행
hook이 호출되면 hook 동작에 따라 handler 실행하고,  
httpGet, tcpSocket은 kubelet 프로세스에 의해 실행되고, exec는 컨테이너에서 실행됨.

#### 보장
일반적으로 여러번 호출되어도 전달은 단 한번만 이루어짐.

#### debugging hook handler
로그는 pod event로 노출되지 않음  
> 이벤트 발송 로그 보기  
> kubectl describe pod <파드_이름>
> 
```bash
Events:
  FirstSeen  LastSeen  Count  From                                                   SubObjectPath          Type      Reason               Message
  ---------  --------  -----  ----                                                   -------------          --------  ------               -------
  1m         1m        1      {default-scheduler }                                                          Normal    Scheduled            Successfully assigned test-1730497541-cq1d2 to gke-test-cluster-default-pool-a07e5d30-siqd
  1m         1m        1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Pulling              pulling image "test:1.0"
  1m         1m        1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Created              Created container with docker id 5c6a256a2567; Security:[seccomp=unconfined]
  1m         1m        1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Pulled               Successfully pulled image "test:1.0"
  1m         1m        1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Started              Started container with docker id 5c6a256a2567
  38s        38s       1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Killing              Killing container with docker id 5c6a256a2567: PostStart handler: Error executing in Docker Container: 1
  37s        37s       1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Killing              Killing container with docker id 8df9fdfd7054: PostStart handler: Error executing in Docker Container: 1
  38s        37s       2      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}                         Warning   FailedSync           Error syncing pod, skipping: failed to "StartContainer" for "main" with RunContainerError: "PostStart handler: Error executing in Docker Container: 1"
  1m         22s       2      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Warning   FailedPostStartHook
```