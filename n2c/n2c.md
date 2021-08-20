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
