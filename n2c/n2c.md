# N2C

Naver에서 제공하는 kubernetes

# L4 Load Balancer와 Ingress

## DNS

LoadBalancer 의 VIP 는 아래 룰로 Discovery 할 수 있습니다.  
[service-name].[namespace].svc.[cluster-name].io.navercorp.com

# 서비스 배포 전략

## Deployment의 pod 개수 변경

- replica 수 변경 시 Deployment.yml 변경 없이

```bash
kubectl scale --replicas 3 deploy http-server-deploy-albamc
```

와 같이 수동으로 적용 가능

- 부하가 많아지면 pod 개수 변경하도록 설정 가능

## Rolling Update

- kubectl에서 기본 제공하는 방식
- v1 -> v2버전으로 배포 시, Deployment.yml에서 이미지 변경하게되면,  
  replicationset을 추가하고 pod을 순차적으로 교체해준다.

## Progressive

blue/green 배포 시, blue와 green 스위칭 할 때, 한개씩 순차적으로 스위칭하면서 pod 개수를 일정하게 유지한다.

## Blue/Green 배포

보통 실제 운영에 적용 시에는  
weight(가중치)를 설정하여 스위칭 시, blue/green의 pod 비율을 맞추어 스위칭하게된다.

```yml
kind: Service
apiVersion: v1
metadata:
  name: http-service-${USER}
  annotations:
    kube-router.io/default-weights: "0"
    kube-router.io/weights.http-blue-${USER}: "10"
    kube-router.io/weights.http-green-${USER}: "90"
spec:
  selector:
    app.kubernetes.io/instance: http-service-${USER}
  ports:
    - protocol: TCP
      port: 80
```

# 모니터링

n2c 클러스터에 namespace를 생성하면, 모니터링에 필요한 각종 컴포넌트가 자동으로 생성

- prometheus : 모니터링 데이터를 모음
- grafana : 대시보드에 그래프 형태로 Drawing

## 사용자 정의 대시보드
pod annotation에 prometeus로 추가 수집할 metric을 설정할 수 있다.
```yml
annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "{exporter port}"
    prometheus.io/path: "{exporter metric path}"
```

# 스토리지
- Local : Emphemeral만 제공하고 Local volume 미제공
- Remote
  - Nobes Fuse : 여러 pod에서 share가능한 공유 스토리지
  - Ceph RBD : pod에서 1:1 매핑되는 고성능 스토리지

# 오토스케일
- 평균 cpu 사용량에 따른 오토스케일러 제공


# HELM CHART
- 설정을 변수로 빼서 yml 관리를 쉽게 함

# 리얼 서비스 적용
## 도커 보안 검수
- 리얼 환경에는 ncc docker push를 직접 할 수 없음
- promote를 이용해 보안검수에 만족하는 경우에만 배포 가능

## ACL 자동화
- Pod을 생성하면 사내에서 접근 가능한 IP를 부여하는 이유가 ACL 때문
- Pod이 생성되면 ACL 포탈에 해당 Pod ACL 자동 생성

## Pod LifeCycle
- initContainer : Pod.yml 설정을 사용해 pod 뜨기 전후로 어떤 작업을 자동으로 할 수 있음
  ```yml
  apiVersion: v1
  kind: Pod
  metadata:
    name: myapp-pod
    labels:
      app: myapp
  spec:
    containers:
    - name: myapp-container
      image: base.registry.navercorp.com/alpine:3.8
      command: ['sh', '-c', 'echo The app is running! && sleep 3600']
    initContainers:
    - name: init-myservice
      image: base.registry.navercorp.com/alpine:3.8
      command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
    - name: init-mydb
      image: base.registry.navercorp.com/alpine:3.8
      command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
  ```
- Container hooks : container 시작/종료 전후로 추가적인 작업 설정 가능.  
  Pod.yml에 설정
  ```yml
  apiVersion: v1
  kind: Pod
  metadata:
    name: lifecycle-demo
  spec:
    containers:
    - name: lifecycle-demo-container-command
      image: nginx
      lifecycle:
        postStart:
          exec:
            command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
        preStop:
          exec:
            command: ["/usr/sbin/nginx","-s","quit"]
    - name: lifecycle-demo-container-http
      image: nginx
      lifecycle:
        postStart:
          httpGet:
            scheme: HTTP
            host: events.container.navercorp.com
            port: 33010
            path: /container-start
            httpHeaders:
            - name: x-token:
              value: ciepndxn
        preStop:
          httpGet:
            scheme: HTTP
            host: events.container.navercorp.com
            port: 33010
            path: /container-stop
            httpHeaders:
            - name: x-token
              value: ciepndxn
  ```
- Probe (Pod상태 체크) : pod의 상태 체크를 통해 pod 자동 재시작 혹은 l4 등록/제거 설정
  ```yml
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      test: liveness
    name: liveness-http
  spec:
    containers:
    - name: liveness
      args:
      - /server
      image: k8s.gcr.io/liveness
      livenessProbe:
        httpGet:
          # when "host" is not defined, "PodIP" will be used
          # host: my-host
          # when "scheme" is not defined, "HTTP" scheme will be used. Only "HTTP" and "HTTPS" are allowed
          # scheme: HTTPS
          path: /healthz
          port: 8080
          httpHeaders:
          - name: X-Custom-Header
            value: Awesome
        initialDelaySeconds: 15
        timeoutSeconds: 1
      readinessProbe:
        tcpSocket:
          port: 8080
  ```

  ## 기타 주의사항
  - 서비스를 생성할 때, IP를 자동으로 발급받는데, VIP를 사용하는 서비스의 삭제로 인해 장애가 나는 것을 방지
    -  ncc.navercorp.com/delete-protected 설정
      - 해당 설정 없어도 내부적으로 30분 ~ 1시간가량 삭제하지 않도록 되어있음
      - 동일한 서비스 생성할 경우 동일 IP 부여

