# cluster architecture
### node
- 컨테이너를 pod 내에 배치하고 node에서 실행함으로써 workload를 구동한다.
- 각 node는 control plane에 의해 관리된다.
- node의 컴포넌트에는 kubelet, container runtime, kube-proxy가 포함됨
  > kubelet : container 각 노드에서 실행되는 agent  
  > container runtime : container 실행을 담당하는 소프트웨어  
  > kube-proxy : 클러스터 각 노드에서 실행되는 network proxy  

### node 추가하기
- node의 kubelet으로 control plane에 자체 등록
  ```json
  {
    "kind": "Node",
    "apiVersion": "v1",
    "metadata": {
      "name": "10.240.79.157", // 유효한 DNS 서브도메인 이름이어야 함
      "labels": {
        "name": "my-first-k8s-node"
      }
    }
  }
  ```
  - --kubeconfig : api 인증 경로
  - --cloud-provider : 클라우드 제공자와의 소통 방법
  - --register-node : 자동으로 api 서버에 등록
  - --register-with-taints : taint(\<key>=\<value>:\<effect>) 리스트 노드 등록
  - --node-ip : node의 ip주소
  - --node-labels : 등록할 노드에 대한 label
  - --node-status-update-frequency : kubelet이 master에 node 상태를 게시할 주기

- 사용자가 node object 수동 추가

# control plane - node 간 통신
- hub and spoke API pattern : 대도시 터미널 집중 방식(?)
- 원격 서비스를 노출하지 않음
- HTTPS로 통신하고 HTTPS end-point로 redirection되는 가상 IP 주소(kube-proxy)로 구성

### API서버에서 kubelet으로의 통신
- 용도
  - pod에 대한 로그를 가져옴
  - 실행중인 pod에 연결
  - kubelet의 port forwarding 기능 제공

- 유의점
  - 기본적으로 API 서버는 kubelet의 serving 인증서 확인을 안하므로 안전하지 않음
  - 따라서 필요 시, ssh 터널링 사용
  - 필요 시, kubelet 인증/권한 부여 활성화

### API서버에서 node/pod 및 서비스로의 통신
기본적으로 HTTP연결되어 암호화되지 않는다.
- ssh tunneling
  - 노드의 통신 경로 보호하는 역할
  - konnectivity로 대체됨

# controller
- 클러스터의 상태를 관찰한 다음, 필요한 경우에 생성/변경 요청을 하는 컨트롤 루프
- 각 컨트롤러는 의도한 상태에 가깝게 이동

# controller 실행하는 방법 : kube-controller-manager
- 클라우드 controller manager = 쿠버네티스 control plane component

### node controller
node object를 `생성`하는 역할
- node object `초기화`
- 클라우드 관련 정보를 사용해 node object에 annotation, label 작성
- node의 호스트 이름과 네트워크 주소 가져옴
- node `상태 확인`

### route controller
사용자의 클러스터의 다른 노드에 있는 각각의 컨테이너가 서로 통신할 수 있도록 route를 적절히 구성해야 함.
필요 시, pod network IP주소 블록 할당

### service controller
클라우드 공급자 API와 상호 작용하여 필요한 서비스 리소스를 선언할 때, 로드 밸런서와 기타 infra structure component 설정
