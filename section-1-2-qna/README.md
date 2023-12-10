> Master node의 구성 요소 4가지

kube-apiserver, controller manager, scheduler, etcd cluster

> Worker node의 구성 요소 3가지

kubelet, container runtime engine, kube-proxy

> `kubectl run nginx --image=nginx` 를 수행할 때 kube apiserver에서 수행하는 프로세스

authenticate user, validate request, retrieve & update data in etcd data store, interact with scheduler/kubelet 

> Node controller의 node monitor period, node monitor grace period, pod eviction timeout의 의미

node monitor period - Node controller가 각 node를 monitor하는 주기 (health check 주기)
node monitor grace period - health check에 실패했을 때, grace period만큼 wait한 뒤에 해당 node를 unreachable하다고 marking한다
pod eviction timeout - ~만큼 unreachable한 node가 되살아나길 기다림. (해당 node의 pod들이 Replica set의 일부일 경우) 그 시간 이후에도 응답이 없으면 해당 node에 있던 pod들을 제거하고, 다른 reachable node에 pod를 새로 생성한다. 

> Pod를 실제로 배치하는 역할

master node의 scheduler에서는 특정 pod를 어느 노드에 배치할지, pod의 resource requirement를 기준으로 node들을 filter&rank할 뿐. 실제로 배치하는 것은 배정된 worker node에 있는 kubelet.

> Replication Controller과 ReplicaSet의 차이점

config 상에서 version이 RC는 v1, RS는 apps/v1.
RS는 selector(spec.selector) 정의가 필수로 필요함. Fail하는 pod가 생길 경우, 어떤 스펙의 pod를 생성할지 알아야 함.

> Service의 종류 3가지  

NodePort, ClusterIP, LoadBalancer



> NodePort service를 configure하려면 필요한 3가지 포트

Target port, port, NodePort

이 중 반드시 명시해야 하는 것은 `Port`.
`Port` - Service(Node 내의 일종의 virtual server)의 port  
`Target port` - Node 내에 접근하고자 하는 pod의 port (명시 안 해주면 Port랑 같게 지정됨)  
`Node port` - Node의 port (명시 안 해주면 30,000 ~ 32,767 중에 남는 것으로 랜덤 할당)

> Node의 IP 198.160.10.1, cluster의 ip 10.100.11.12, pod의 ip 10.222.0.2, NodePort가 30080, port 80, target port 80일 때 로컬에서 curl로 pod에 접근하려면?

curl http://198.160.10.1:30080

> Service에서 연결시킬 pod를 명시하기 위해 사용하는 필드?

service.yaml의 spec.selector 하위 필드는 pod.yaml의 metadata.labels 하위와 같다.

> LoadBalancer service가 NodePort service와 똑같이 동작하는 상황은?

LoadBalancer service를 네이티브하게 지원하지 않는 cloud service provider(ex. VirtualBox) 환경인 경우

> k8s object configuration yaml file 에 명시하는 최상위 필드 4가지

apiVersion, kind, metadata, spec

> kubectl apply 커맨드를 실행할 때 고려되는 세 가지 파일

1) local configuration file(로컬에 저장됨)  
2) live object configuration(k8s 메모리에 저장됨)  
3) last applied configuration(local config yaml이 json 포맷으로 변환된 뒤, live object config의 annotation으로 명시됨. 즉, k8s 메모리에 저장됨)

last applied config랑 local config를 비교해서, 이 둘이 다르면 live object config를 업데이트하는 식.  

> 자동적으로 생성되는 kuberntetes namespace 3가지

default, kube-system, kube-public

> dev namespace 에 redis pod를 생성하도록 하는 imperative command

`kubectl run redis --image=redis --namespace=dev`

> Imperative/Declarative command의 차이점

imperative - step을 명시 / declarative - 최종 desired 상태를 정의

> ReplicaSet을 삭제/업데이트하면 pod는 어떻게 되는지?

RS를 삭제하면 pod들도 삭제된다. RS가 업데이트되면 그대로 있다가, 기존 pod들을 삭제해야 업데이트된 스펙의 pod들이 생성된다.

> --dry-run 의미
실제로 수행하지는 않고 시뮬레이션

> namespace에 자원 할당을 제한하고 싶을 때 활용하는 k8s object

ResourcQuota (meta.namespace, spec.hard 명시)

> imperative하게 deployment 생성하기

`kubectl create deployment nginx --image=nginx --replicas=6`

> imperative하게 pod를 생성하고 port를 서비스를 통해 expose하는 법

`kubectl run redis --image=redis --port=80 --expose`

> imperative하게 NodePort service 생성하기

`kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service`
이렇게 하면 node port를 명시하진 못해서, 위 커맨드에 --dry-run -o yaml > temp.yaml 해서 config 파일 생성한 뒤에 node port 명시해줘야. 또는

`kubectl create service nodeport nginx --tcp 80:80 --node-port=30080` 이런식으로 node port 명시는 가능하지만, 위 커맨드처럼 pod를 selector로 명시하진 못함. (위에서 pod selector가 nginx)

> 특정 namespace에 있는 pod 조회하기

`kubectl get pods --namespace=dev`
`kubectl get pods --all-namespace`