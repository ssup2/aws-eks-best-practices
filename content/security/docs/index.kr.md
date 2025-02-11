# 보안을 위한 Amazon EKS 모범 사례 가이드
이 가이드는 위험 평가 및 완화 전략을 통해 비즈니스 가치를 제공하는 동시에 EKS에 의존하는 정보, 시스템 및 자산을 보호하는 방법에 대한 조언을 제공합니다. 여기의 지침은 고객이 모범 사례에 따라 EKS를 구현하는 데 도움이 되도록 AWS에서 게시하는 일련의 모범 사례 가이드 중 일부입니다. 성능, 운영 우수성, 비용 최적화 및 안정성에 대한 가이드는 앞으로 몇 개월 내에 제공될 예정입니다.

## 이 가이드를 사용하는 방법
이 가이드는 EKS 클러스터와 클러스터가 지원하는 워크로드에 대한 보안 제어의 효과를 구현하고 모니터링하는 책임이 있는 보안 실무자를 대상으로 합니다. 가이드는 더 쉽게 사용할 수 있도록 다양한 주제 영역으로 구성되어 있습니다. 각 주제는 간략한 개요로 시작하여 EKS 클러스터 보안을 위한 권장 사항 및 모범 사례 목록이 이어집니다. 주제는 특정 순서로 읽을 필요가 없습니다.

## 공동 책임 모델 이해
보안 및 규정 준수는 EKS와 같은 관리형 서비스를 사용할 때 공동 책임으로 간주됩니다. 일반적으로 말해서 AWS는 클라우드 "의" 보안을 책임지고 고객인 귀하는 클라우드 "안"의 보안을 책임집니다. EKS에서 AWS는 EKS 관리형 Kubernetes 컨트롤 플레인 관리를 담당합니다. 여기에는 Kubernetes 마스터, ETCD 데이터베이스 및 AWS가 안전하고 안정적인 서비스를 제공하는 데 필요한 기타 인프라가 포함됩니다. EKS의 소비자는 IAM, 포드 보안, 런타임 보안, 네트워크 보안 등과 같은 이 가이드의 주제에 대해 주로 책임이 있습니다.

인프라 보안과 관련하여 AWS는 자체 관리 작업자에서 관리형 노드 그룹, Fargate로 이동할 때 추가 책임을 맡습니다. 예를 들어 Fargate를 사용하면 AWS가 Pod를 실행하는 데 사용되는 기본 인스턴스/런타임을 보호할 책임이 있습니다.

![ 공동 책임 모델 - Fargate ]( images/SRM-EKS.jpg )

AWS는 또한 Kubernetes 패치 버전 및 보안 패치로 EKS 최적화 AMI를 최신 상태로 유지할 책임이 있습니다. MNG(관리형 노드 그룹)를 사용하는 고객은 EKS API, CLI, Cloudformation 또는 AWS 콘솔을 통해 노드 그룹을 최신 AMI로 업그레이드할 책임이 있습니다. 또한 Fargate와 달리 MNG는 인프라/클러스터를 자동으로 확장하지 않습니다. [ cluster- autoscaler ] ( https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md ) 또는 [ Karpenter ]( https ://karpenter.sh/ ), 기본 AWS 자동 확장, SpotInst의 [ Ocean ]( https://spot.io/solutions/kubernetes-2/ ) 또는 Atlassian의 [ Escalator ]( https://github.com/atlassian/ 에스컬레이터 ).

![ 공유 책임 모델 - MNG ]( ./images/SRM-MNG.jpg )

시스템을 설계하기 전에 귀하의 책임과 서비스 공급자(AWS) 사이의 경계선이 어디인지 아는 것이 중요합니다.

공동 책임 모델에 대한 추가 정보는 [ https://aws.amazon.com/compliance/shared-responsibility-model/ ]( https://aws.amazon.com/compliance/shared-responsibility-model/ ) 을 참조하십시오.

## 소개
EKS와 같은 관리형 Kubernetes 서비스를 사용할 때 관련된 몇 가지 보안 모범 사례 영역이 있습니다.

+ ID 및 액세스 관리
+ 포드 보안
+ 런타임 보안
+ 네트워크 보안
+ 멀티테넌시
+ 탐정 컨트롤
+ 인프라 보안
+ 데이터 암호화 및 비밀 관리
+ 규정 준수
+ 사고 대응 및 법의학
+ 이미지 보안

시스템 설계의 일부로 보안 상태에 영향을 미칠 수 있는 보안 관련 사항과 관행에 대해 생각해야 합니다. 예를 들어 리소스 집합에 대해 작업을 수행할 수 있는 사람을 제어해야 합니다. 또한 보안 사고를 신속하게 식별하고 무단 액세스로부터 시스템과 서비스를 보호하며 데이터 보호를 통해 데이터의 기밀성과 무결성을 유지할 수 있는 능력이 필요합니다. 보안 사고에 대응하기 위해 잘 정의되고 연습된 일련의 프로세스를 갖추면 보안 태세도 향상됩니다. 이러한 도구와 기술은 재정적 손실 방지 또는 규제 의무 준수와 같은 목표를 지원하기 때문에 중요합니다.

AWS는 보안에 민감한 다양한 고객의 피드백을 기반으로 진화한 다양한 보안 서비스 세트를 제공하여 조직이 보안 및 규정 준수 목표를 달성할 수 있도록 지원합니다. 매우 안전한 기반을 제공함으로써 고객은 "차별화되지 않은 무거운 작업"에 소요되는 시간을 줄이고 비즈니스 목표를 달성하는 데 더 많은 시간을 할애할 수 있습니다.

## 피드백
이 가이드는 광범위한 EKS/Kubernetes 커뮤니티에서 직접적인 피드백과 제안을 수집하기 위해 GitHub에서 릴리스되고 있습니다. 가이드에 포함해야 한다고 생각하는 모범 사례가 있는 경우 GitHub 리포지토리에 문제를 제출하거나 PR을 제출하세요. 우리의 의도는 서비스에 새로운 기능이 추가되거나 새로운 모범 사례가 발전할 때 가이드를 주기적으로 업데이트하는 것입니다.

## 더 읽을 거리
[ Kubernetes 보안 백서 ]( https://github.com/kubernetes/sig-security/blob/main/sig-security-external-audit/security-audit-2019/findings/Kubernetes%20White%20Paper.pdf ), 후원 Security Audit Working Group에서 만든 이 백서는 보안 실무자가 건전한 설계 및 구현 결정을 내리는 데 도움을 주기 위해 Kubernetes 공격 표면 및 보안 아키텍처의 주요 측면을 설명합니다.

CNCF는 또한 클라우드에 [ 백서 ]( https://github.com/cncf/tag-security/blob/efb183dc4f19a1bf82f967586c9dfcb556d87534/security-whitepaper/v2/CNCF_cloud-native-security-whitepaper-May2022-v2.pdf )를 게시했습니다. 기본 보안. 이 백서에서는 기술 환경이 어떻게 발전했는지 살펴보고 DevOps 프로세스 및 애자일 방법론과 일치하는 보안 관행의 채택을 옹호합니다.


