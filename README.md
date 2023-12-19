<h1>신한투자증권 ICT기획운영부 개인과제</h1>
- 목표 : 금융 보안 정책 기반 클라우드 아키텍처 설계 및 서비스 제공 환경 구현 <br>

## 아키텍처 및 전체 구성

![image](https://github.com/2522001/shinhan-ict/assets/83651335/9371b5d4-0182-443a-96df-798600031ef9)

## 과제 목표 및 진행 일정

![image](https://github.com/2522001/shinhan-ict/assets/83651335/726f3275-e777-4dfb-8a62-62a65edb5049)

### 10/30~11/3

이 기간동안 두 가지 방법으로 쿠버네티스를 구축해보았고, Jenkins를 활용하여 CI 파이프라인을 구축했습니다

초기 과제 목표는 쿠버네티스보다 CI/CD에 초점이 맞추어져 있었습니다. 백엔드 개발에 관심이 많은 만큼, 이 분야에서도 자주 활용할 수 있는 CI/CD 부분에 집중하고자 했습니다. 따라서 초반에 쿠버네티스 및 CI/CD 개념에 대한 전반적인 공부를 한 후 NKS를 활용하여 쿠버네티스를 구축했고, 여기에 Jenkins와 ArgoCD를 활용하여 CI/CD를 진행하고자 했습니다.

<p align="center">
  <img src="https://github.com/2522001/shinhan-ict/assets/83651335/6ba48fe6-3ef9-4c27-b9f8-eaee75254a02" width="600">
</p>

위 그림은 제가 참고한 CI/CD 파이프라인입니다. 개발자가 Dev Git에 push한 내용을 Jenkins가 감지한 후 도커 허브에 이미지를 푸시하고 manifest Git 안의 manifest 파일을 수정하면, ArgoCD는 이를 감지하여 클러스터의 manifest와 Git의 manifest를 비교한 후 불일치 시 Deploy합니다.

그런데 CI까지는 어찌저찌 진행하였으나, 쿠버네티스 내에서 동작하는 ArgoCD로 진입하자 쿠버네티스에 대한 얕은 이해도가 장벽이 되었습니다. 제대로 된 이해없이 하다보니 막히기 일쑤였고, 쿠버네티스를 깊게 공부해보고자 직접 노드를 구성하여 쿠버네티스를 구축해보기로 했습니다.

![image](https://github.com/2522001/shinhan-ict/assets/83651335/9be2ee74-530e-4de1-9faf-4ca05b9b05a4)

하나의 VPC 안에 public subnet, private subnet을 만들고, public subnet에는 bastion 서버를, private subnet에는 master, worker-01, worker-02 노드를 구축했습니다. private subnet에 위치한 노드가 외부와 통신할 수 있게끔 public subnet에 internet gateway와 NAT gateway를 붙였습니다.

**학습하며 작성한 관련 글**

- 쿠버네티스란?
- NKS 구축하기
- 클라우드 환경에 쿠버네티스 직접 구축하기
- CI/CD란?
- CI: Jenkins 설치하기
- Jenkins k8s manifest update stage 실패 이슈 (미해결)

### 11/4~11/6

이 기간동안 외부에서 쿠버네티스 내 서비스에 접근하기 위한 다양한 방법(NodePort, LoadBalancer, Ingress)을 공부하고 구현했습니다.

외부에서 쿠버네티스 내 서비스에 접근하기 위한 방법은 크게 세가지로 나뉩니다.

| 서비스 타입 | 접근 경로 |
| --- | --- |
| NodePort | <서비스가 위치한 워커노드 IP>:<서비스의 노드포트> |
| LoadBalancer | <서비스의 External IP>:<서비스의 포트> |
| Ingress | <Ingress의 주소>:<Nginx-Ingress-Controller의 노드포트>/<서비스의 path>  |

public이 아닌 private 환경에 구성하다보니 막히는 부분이 많았습니다. 워커노드 IP로 바로 외부에서 접근할 수 없고, nks를 사용할 때와 달리 서비스의 타입을 LoadBalancer로 했을 때 External IP도 자동으로 붙지 않았습니다. Ingress의 주소도 마찬가지로 외부에서 접근할 수 없는 주소였습니다.

여러 시행착오를 거쳐 다음과 같이 해결했습니다. 핵심은 NHN Cloud에서 직접 로드밸런서를 만들고, 로드밸런서 포트와 인스턴스 포트를 알맞게 잘 열어주어 쿠버네티스 내 리소스와 연결하는 것입니다.

| 서비스 타입 | NHN Cloud 로드밸런서와 연결하는 방식 |
| --- | --- |
| NodePort | NHN Cloud 로드밸런서의 포트를 서비스의 포트와 동일하게 설정합니다. (ex; 8080, 31634) |
| LoadBalancer | 마스터 노드에 MetalLB를 구성한 후 IP 대역에 NHN Cloud 로드밸런서의 Floating IP를 추가합니다. 서비스의 External IP가 해당 Floating IP로 설정됩니다. |
| Ingress | Nginx-Ingress-Controller를 NodePort 타입으로 설정하고, NHN Cloud 로드밸런서의 인스턴스 포트를 컨트롤러의 노드 포트로 설정합니다. 이때 로드밸런서의 상태 확인 프로토콜은 TCP이어야 합니다. |

다음은 LoadBalancer 방식으로 띄운 nginx 페이지입니다. NodePort나 LoadBalancer의 단점은 서비스 자체를 외부와 연결하는 방식이기 때문에 각 서비스마다 새로운 로드밸런서를 붙여주어야 한다는 점입니다.

<p align="center">
  <img src="https://github.com/2522001/shinhan-ict/assets/83651335/609981fe-3c57-4d7f-84f5-60d3eaa77369" width="600">
</p>

반면 Ingress는 하나의 로드밸런서 IP로 여러가지 서비스에 접근이 가능하기 때문에 사용자 입장에서 편리합니다. 다음은 하나의 IP로 tea, coffee라는 두가지 파드로 접속하는 예제입니다. 각 서비스의 파드는 워커 노드 두 개에 분산되어 있어 Round-Robin 방식으로 작동합니다. 이는 새로고침 시마다 바뀌는 Server address 및 Server name으로 확인 가능합니다.

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/2522001/shinhan-ict/assets/83651335/f7afc400-4478-424b-a5ce-17835c69d247" alt="image5">
    </td>
    <td align="center">
      <img src="https://github.com/2522001/shinhan-ict/assets/83651335/8d0d0d2b-9e11-4910-9263-17533d6c2e88" alt="image6">
    </td>
  </tr>
</table>

**학습하며 작성한 관련 글**

- Ingress란?
- Ingress 설정 및 요청하기
- 클라우드 환경에 쿠버네티스 직접 구축하기
- 상황 별로 ping 및 curl이 보내지지 않는 이슈 (해결)
- NodePort 타입의 서비스와 NHN 로드밸런서 연결하기
- LoadBalancer 타입의 서비스와 NHN 로드밸런서 연결하기
- Ingress와 NHN 로드밸런서 연결하기

### 11/6~11/7

이 기간동안 Jenkins와 ArgoCD를 활용한 CI/CD와 프로메테우스와 그라파나를 활용한 모니터링을 구축하고자 노력했습니다.

**CI: Jenkins**

bastion 서버에 docker 이미지를 통해서 설치했습니다. 이 과정에서 도커 컨테이너 안에 도커를 설정하는 재미있는 경험을 했습니다. 젠킨스 컨테이너 내에서 도커 관련 명령을 수행하려면 docker daemon의 도움이 필요하고, 도커 위에 도커 컨테이너를 생성하는 Docker In Docker 방식과 로컬 머신에서 이미 실행 중인 도커 데몬을 이용하는 Docker out of Docker 방식이 존재합니다. 저는 DInD 방식을 사용했는데, 찾아보니 호스트 컨테이너가 호스트 머신에서 가능한 거의 모든 작업을 할 수 있게 되는 보안 위험이 있다고 하여, 다음 번엔 DooD 방식을 사용하거나 도커가 아닌 다른 방식을 통해 Jenkins를 설치할 것 같습니다.

파이프라인은 개발 관련 git에 체크 아웃 - 도커 이미지 빌드 - 도커 허브에 이미지 푸시 - 운영 관련 git에 manifest 파일 업데이트하는 단계로 구성했습니다. 개발자가 변경 사항을 git에 push하면 github webhook가 이를 알리고, Jenkins는 이미지를 빌드하고 푸시합니다. 도커 허브에 이미지가 푸시되어 이미지 태그가 변경되면, sed 명령어를 통해 운영 git에 있는 manifest 파일의 이미지 태그 부분을 수정해줍니다.

다른 스테이지는 비교적 순조롭게 진행했으나, manifest를 업데이트하는 마지막 스테이지에서 첫 주에 진행했을 당시와 동일한 이슈를 겪었습니다. 운영 git에 add, commit까지는 잘 되는데 push에서 무한 로딩이 걸리는 이슈로, 에러 메시지가 뜨지 않고 레퍼런스도 찾지 못하여 결국 해결하지 못했습니다. git Credential 문제로 가정하고 여러가지 방식으로 인증을 해보고 토큰을 다시 만들어도 보는 등 다양한 시도를 했으나 계속 같은 이슈가 발생합니다.

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/2522001/shinhan-ict/assets/83651335/04fa4d4f-01f7-4a1a-aab5-045264ca1418" alt="image3">
    </td>
    <td align="center">
      <img src="https://github.com/2522001/shinhan-ict/assets/83651335/2f25ef1f-7614-4d08-8870-0788739f9293" alt="image4">
    </td>
  </tr>
</table>

**CD: ArgoCD**

master 노드에 kubectl apply 명령어를 통해 설치했습니다. ArgoCD의 서비스를 로드밸런서 타입으로 설정해주었고, MetalLB를 활용하여 External IP를 부여했습니다. 443포트를 열어주니 페이지는 잘 열렸으나, git connection에 실패하는 이슈가 발생했습니다. 이것도 git Credential 관련 문제인가 싶어서 public github도 연결해보고, HTTP 방식이 문제인가 싶어 SSH 방식도 활용해보았지만 실패했습니다.

ArgoCD Server의 파드 로그를 확인해보니, Argo-Redis 부분에서 timeout 오류가 발생하고 있었습니다. redis 관련 서비스 및 파드를 다시 설치해보고 보안그룹에서 redis의 포트를 열어주는 등 다양한 시도를 해보았으나 여전히 실패했습니다. 쿠버네티스와 ArgoCD 간 버전 호환성 문제일 가능성이 높다고 하는데 아마 이 문제가 아닐까 싶습니다.

<p align="center">
  <img src="https://github.com/2522001/shinhan-ict/assets/83651335/1a1bb508-fe6a-4595-acae-102dfd953a1c" width="600">
</p>

**CI/CD: Jenkins**

Jenkins가 운영 관련 git에 manifest를 업데이트하면 ArgoCD가 클러스터 내부의 manifest와 git의 manifest를 비교한 후, 불일치 시 도커 허브에서 이미지를 pull 해온 후 쿠버네티스에 배포하는 흐름을 구현하고 싶었는데, 앞서 언급한 이슈로 인해 일단은 간소화하기로 했습니다.

ArgoCD를 사용하지 않고, Jenkins가 이미지를 빌드하고 바로 쿠버네티스에 배포하도록 파이프라인을 수정했습니다.

**모니터링: Prometheus & Grafana**

모니터링을 위한 프로메테우스와 그라파나는 Helm이라는 새로운 방식으로 설치해보았습니다. 설치 후 그라파나 파드가 돌아가지 않는 이슈가 있었는데, 원인은 PersistentVolume (PV) 및 PersistentVolumeClaim (PVC)가 설정되지 않은 것이 원인이었습니다. PVC와 PV를 설정할 경우 파드가 죽어도 데이터를 유지할 수 있습니다. 저는 빨리 대시보드를 띄우기 위해서 우선 PV 및 PVC를 사용하지 않도록 설정했는데, 모니터링은 지속적인 데이터가 중요한만큼 개선할 예정입니다.

그런데 여기서도 현재까지 해결하지 못한 이슈가 발생했습니다. 각 대시보드에 접속까지는 문제 없이 되는데, 프로메테우스가 데이터를 가지고 오지 못하고 있고, 이에 따라 그라파나에도 모든 결과가 N/A로 뜹니다. 이 문제는 시간 관계 상 해결하기 위한 시도를 거의 하지 못해서 추후 더 알아볼 예정입니다.

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/2522001/shinhan-ict/assets/83651335/b804f5a2-3556-4336-a9ae-b3018a679896" alt="image1">
    </td>
    <td align="center">
      <img src="https://github.com/2522001/shinhan-ict/assets/83651335/f092216b-392a-4e66-adc0-d0fecac2e9d0" alt="image2">
    </td>
  </tr>
</table>

**학습하며 작성한 관련 글**

- Jenkins & ArgoCD 설치하기
-  CI/CD 이슈 (미해결)
- Helm으로 Prometheus & Grafana 설치하기
- NKS에 모니터링 구축하기

## 느끼고 배운점

맨 땅에 헤딩하는 방식으로 접근하다보니 투자한 시간에 비해 결과가 아쉽지만, 성장이라는 뿌듯함을 느꼈습니다. 저는 클라우드에 대한 정말 기초적인 지식만 가지고 있었고, 처음에 쿠버네티스가 무엇인지 공부할 때만 해도 어렴풋이 이해만 했을 뿐 관련 개념이 잘 그려지지 않았습니다. 일주일이 조금 넘는 시간 동안 쿠버네티스가 무엇인지, CI/CD는 어떻게 이루어지는지, 외부에서 private환경에 접근하려면 어떻게 해야하는지 등 하나씩 알아갔고, 지금은 쿠버네티스를 활용한 개발 및 운영 시스템이 어떻게 이루어지는지 큰 그림을 그릴 수 있게 되었습니다.

이 기회가 아니었으면 평생 하지 않았을지도 모르는 분야의 공부를 깊게 해보고, 나아가 인프라 및 클라우드라는 새로운 흥미를 가지게 된 것에 감사합니다.
