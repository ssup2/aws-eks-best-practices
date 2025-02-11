# 클러스터 업그레이드 모범 사례

이 안내서는 클러스터 관리자에게 Amazon EKS 업그레이드 전략을 계획하고 실행하는 방법을 보여줍니다.또한 자체 관리형 노드, 관리형 노드 그룹, 카펜터 노드 및 Fargate 노드를 업그레이드하는 방법도 설명합니다.EKS Anywhere, 자체 관리형 쿠버네티스, AWS 아웃포스트 또는 AWS 로컬 영역에 대한 지침은 포함되지 않습니다. 

## 개요

Kubernetes 버전은 컨트롤 플레인과 데이터 플레인을 모두 포함합니다.원활한 작동을 위해 컨트롤 플레인과 데이터 플레인 모두 동일한 [쿠버네티스 마이너 버전, 예: 1.24](https://kubernetes.io/releases/version-skew-policy/#supported-versions) 를 실행해야 합니다. 

* **컨트롤 플레인** — 컨트롤 플레인 버전은 쿠버네티스 API 서버에서 정의합니다.EKS 클러스터에서는 AWS에서 이를 관리합니다.컨트롤 플레인 버전으로의 업그레이드는 AWS API를 사용하여 시작됩니다. 
* **데이터 플레인** — 데이터 플레인 버전은 노드에서 실행되는 Kubelet 버전을 참조합니다.동일한 클러스터의 여러 노드라도 버전이 다를 수 있습니다. `kubectl get nodes` 명령어로 있는 모든 노드의 버전을 확인하세요. 


## 클러스터를 최신 상태로 유지

Kubernetes 릴리스를 최신 상태로 유지하는 것은 EKS 및 Kubernetes를 채택할 때 공동 책임 모델의 중요한 구성 요소입니다. 

특정 시점 (보통 1년) 이 지나면 쿠버네티스 커뮤니티는 버그 및 CVE 패치 릴리스를 중단합니다.또한 쿠버네티스 프로젝트는 더 이상 사용되지 않는 버전에 대한 CVE 제출을 권장하지 않습니다.즉, 이전 버전의 Kubernetes에만 있는 취약성은 보고조차 되지 않을 수 있으며, 따라서 취약성 발생 시 알림 없이 노출되거나 해결 옵션도 제공되지 않을 수 있습니다.

당사는 이를 EKS 및 당사 고객이 받아들일 수 없는 보안 태세로 간주하여 최신 버전으로 자동 업그레이드하는 현행 정책을 시행하고 있습니다.[EKS 버전 지원 정책 검토](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html).

지원 종료일 이전에 클러스터를 업그레이드하지 않으면 지원되는 다음 버전으로 자동 업그레이드됩니다.[자세한 내용은 버전 FAQ를 참조하십시오.](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html#version-deprecation) 적절한 테스트 및 준비가 없으면 워크로드와 컨트롤러가 중단될 수 있습니다.**EKS의 Kubernetes 릴리스를 최신 상태로 유지하는 것이 좋습니다.**

클러스터 업그레이드 처리를 위한 잘 문서화된 프로세스를 개발하십시오.클러스터 업그레이드를 지원하는 런북과 도구를 빌드하세요. 

클러스터를 정기적으로 업그레이드할 계획을 세우세요. EKS Kubernetes 릴리스를 따라잡으려면 적어도 1년에 한 번 클러스터를 업그레이드하십시오. 

## EKS 출시 일정 검토

[EKS 쿠버네티스 릴리스 캘린더 검토](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html#kubernetes-release-calendar) 를 통해 새 버전이 출시되는 시기와 특정 버전에 대한 지원이 종료되는 시기를 알아보십시오.일반적으로 EKS는 매년 세 개의 마이너 버전의 쿠버네티스를 릴리스하며 각 마이너 버전은 약 14개월 동안 지원됩니다. 

또한 업스트림 [쿠버네티스 릴리스 정보](https://kubernetes.io/releases/) 도 검토하십시오.

## 공동 책임 모델이 클러스터 업그레이드에 어떻게 적용되는지 이해

클러스터 컨트롤 플레인과 데이터 플레인 모두에 대한 업그레이드를 시작하는 것은 사용자의 책임입니다.[업그레이드 시작 방법에 대해 알아보십시오.](https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html) 클러스터 업그레이드를 시작하면 AWS가 클러스터 컨트롤 플레인 업그레이드를 관리합니다.Fargate 파드 및 [기타 애드온] 을 포함한 데이터 플레인 업그레이드는 사용자의 책임입니다.(#upgrade -애드온 및 구성 요소 - kubernetes-api 사용) 클러스터 업그레이드 후 가용성과 운영에 영향을 미치지 않도록 클러스터에서 실행되는 워크로드의 업그레이드를 검증하고 계획해야 합니다.

## 클러스터를 인플레이스 업그레이드

EKS는 인플레이스 클러스터 업그레이드 전략을 지원합니다.이렇게 하면 클러스터 리소스가 유지되고 클러스터 구성 (예: API 엔드포인트, OIDC, ENIS, 로드 밸런서) 이 일관되게 유지됩니다.이렇게 하면 클러스터 사용자의 업무 중단이 줄어들고, 워크로드를 재배포하거나 외부 리소스 (예: DNS, 스토리지) 를 마이그레이션할 필요 없이 클러스터의 기존 워크로드와 리소스를 사용할 수 있습니다.

전체 클러스터 업그레이드를 수행할 때는 한 번에 하나의 마이너 버전 업그레이드만 실행할 수 있다는 점에 유의해야 합니다 (예: 1.24에서 1.25까지). 

즉, 여러 버전을 업데이트해야 하는 경우 일련의 순차적 업그레이드가 필요합니다.순차적 업그레이드를 계획하는 것은 더 복잡하며 다운타임이 발생할 위험이 더 높습니다.이 상황에서는 [블루/그린 클러스터 업그레이드 전략을 평가하십시오.](#인플레이스-클러스터-업그레이드의-대안으로-블루/그린-클러스터-평가)

## 컨트롤 플레인과 데이터 플레인을 순서대로 업그레이드

클러스터를 업그레이드하려면 다음 조치를 취해야 합니다.

1.[쿠버네티스 및 EKS 릴리스 노트를 검토하십시오.](#EKS-설명서를-사용하여-업그레이드-체크리스트-생성)
2.[클러스터를 백업하십시오.(선택 사항)](#업그레이드-전-클러스터-백업)
3.[워크로드에서 더 이상 사용되지 않거나 제거된 API 사용을 식별하고 수정하십시오.](#컨트롤-플레인을-업그레이드하기-전에-제거된-API-사용을-식별하고-수정하기)
4.[관리형 노드 그룹을 사용하는 경우 컨트롤 플레인과 동일한 Kubernetes 버전에 있는지 확인하십시오.](#노드의-버전-스큐를-추적하여-업그레이드하기-전에-관리형-노드-그룹이-컨트롤-플레인과-동일한-버전에-있는지-확인) EKS Fargate Profile에서 생성한 EKS 관리형 노드 그룹 및 노드는 컨트롤 플레인과 데이터 플레인 간의 마이너 버전 스큐를 1개만 지원합니다.
5.[AWS 콘솔 또는 CLI를 사용하여 클러스터 컨트롤 플레인을 업그레이드하십시오.](https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html)
6.[애드온 호환성을 검토하세요.](#쿠버네티스-API를-사용하여-애드온-및-컴포넌트-업그레이드) 필요에 따라 쿠버네티스 애드온과 커스텀 컨트롤러를 업그레이드하십시오. 
7.[kubectl 업데이트하기.](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
8.[클러스터 데이터 플레인을 업그레이드합니다.](https://docs.aws.amazon.com/eks/latest/userguide/update-managed-node-group.html) 업그레이드된 클러스터와 동일한 쿠버네티스 마이너 버전으로 노드를 업그레이드하십시오. 

## Upgrade your control plane and data plane in sequence

To upgrade a cluster you will need to take the following actions:

## EKS 설명서를 사용하여 업그레이드 체크리스트 생성

EKS 쿠버네티스 [버전 설명서](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html) 에는 각 버전에 대한 자세한 변경 목록이 포함되어 있습니다.각 업그레이드에 대한 체크리스트를 작성하십시오. 

특정 EKS 버전 업그레이드 지침은 설명서에서 각 버전의 주요 변경 사항 및 고려 사항을 검토하십시오.

* [EKS 1.27](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html#kubernetes-1.27)
* [EKS 1.26](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html#kubernetes-1.26)
* [EKS 1.25](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html#kubernetes-1.25)
* [EKS 1.24](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html#kubernetes-1.24)
* [EKS 1.23](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html#kubernetes-1.23)
* [EKS 1.22](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html#kubernetes-1.22)

## 쿠버네티스 API를 사용하여 애드온 및 컴포넌트 업그레이드

클러스터를 업그레이드하기 전에 사용 중인 Kubernetes 구성 요소의 버전을 이해해야 합니다. 클러스터 구성 요소의 인벤토리를 작성하고 Kubernetes API를 직접 사용하는 구성 요소를 식별하십시오.여기에는 모니터링 및 로깅 에이전트, 클러스터 오토스케일러, 컨테이너 스토리지 드라이버 (예: [EBS CSI](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html), [EFS CSI](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html)), 인그레스 컨트롤러, 쿠버네티스 API를 직접 사용하는 기타 워크로드 또는 애드온과 같은 중요한 클러스터 구성 요소가 포함됩니다. 

!!! 팁
    중요한 클러스터 구성 요소는 대개 `*-system` 네임스페이스에 설치됩니다.
    
    ```
    kubectl get ns | grep '-system'
    ```

Kubernetes API를 사용하는 구성 요소를 식별한 후에는 해당 설명서에서 버전 호환성 및 업그레이드 요구 사항을 확인하십시오.예를 들어 버전 호환성에 대해서는 [AWS 로드 밸런서 컨트롤러](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/deploy/installation/) 설명서를 참조하십시오.클러스터 업그레이드를 진행하기 전에 일부 구성 요소를 업그레이드하거나 구성을 변경해야 할 수 있습니다.확인해야 할 몇 가지 중요한 구성 요소로는 [CoreDNS](https://github.com/coredns/coredns), [kube-proxy](https://kubernetes.io/docs/concepts/overview/components/#kube-proxy), [VPC CNI](https://github.com/aws/amazon-vpc-cni-k8s), 스토리지 드라이버 등이 있습니다. 

클러스터에는 Kubernetes API를 사용하는 많은 워크로드가 포함되는 경우가 많으며 인그레스 컨트롤러, 지속적 전달 시스템, 모니터링 도구와 같은 워크로드 기능에 필요합니다.EKS 클러스터를 업그레이드할 때는 애드온과 타사 도구도 업그레이드하여 호환되는지 확인해야 합니다.
 
일반적인 애드온의 다음 예와 관련 업그레이드 설명서를 참조하십시오.

* **Amazon VPC CNI:** 각 클러스터 버전에 대한 Amazon VPC CNI 애드온의 권장 버전은 [쿠버네티스 자체 관리형 애드온용 Amazon VPC CNI 플러그인 업데이트](https://docs.aws.amazon.com/eks/latest/userguide/managing-vpc-cni.html) 를 참조하십시오. **Amazon EKS 애드온으로 설치한 경우 한 번에 하나의 마이너 버전만 업그레이드할 수 있습니다.**
* **kube-proxy:** [쿠버네티스 kube-proxy 자체 관리형 애드온 업데이트](https://docs.aws.amazon.com/eks/latest/userguide/managing-kube-proxy.html) 를 참조하십시오.
* **CoreDNS:** [CoreDNS 자체 관리형 애드온 업데이트](https://docs.aws.amazon.com/eks/latest/userguide/managing-coredns.html) 를 참조하십시오.
* **AWS Load Balancer Controller:** AWS 로드 밸런서 컨트롤러는 배포한 EKS 버전과 호환되어야 합니다.자세한 내용은 [설치 가이드](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html) 를 참조하십시오. 
* **아마존 엘라스틱 블록 스토어 (아마존 EBS) 컨테이너 스토리지 인터페이스 (CSI) 드라이버:** 설치 및 업그레이드 정보는 [아마존 EKS 애드온으로 아마존 EBS CSI 드라이버 관리](https://docs.aws.amazon.com/eks/latest/userguide/managing-ebs-csi.html) 를 참조하십시오.
* **아마존 엘라스틱 파일 시스템 (Amazon EFS) 컨테이너 스토리지 인터페이스 (CSI) 드라이버:** 설치 및 업그레이드 정보는 [아마존 EFS CSI 드라이버](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html) 를 참조하십시오.
* **쿠버네티스 메트릭 서버:** 자세한 내용은 GitHub의 [메트릭스-서버](https://kubernetes-sigs.github.io/metrics-server/) 를 참조하십시오.
* **Kubernetes 클러스터 오토스케일러:** Kubernetes 클러스터 오토스케일러 버전을 업그레이드하려면 배포 시 이미지 버전을 변경하십시오.클러스터 오토스케일러는 쿠버네티스 스케줄러와 밀접하게 연결되어 있습니다.클러스터를 업그레이드할 때는 항상 업그레이드해야 합니다.[GitHub 릴리스](https://github.com/kubernetes/autoscaler/releases) 를 검토하여 쿠버네티스 마이너 버전에 해당하는 최신 릴리스의 주소를 찾으십시오.

* **카펜터:** 설치 및 업그레이드 정보는 [카펜터 설명서](https://karpenter.sh/v0.27.3/faq/#which-versions-of-kubernetes-does-karpenter-support)를 참조하십시오.

## 업그레이드 전에 기본 EKS 요구 사항 확인

AWS에서 업그레이드 프로세스를 완료하려면 계정에 특정 리소스가 필요합니다.이러한 리소스가 없는 경우 클러스터를 업그레이드할 수 없습니다.컨트롤 플레인 업그레이드에는 다음 리소스가 필요합니다.

1.사용 가능한 IP 주소.클러스터를 업데이트하려면 Amazon EKS에서 클러스터를 생성할 때 지정한 서브넷의 사용 가능한 IP 주소가 최대 5개까지 필요합니다.
2.EKS IAM 역할: 컨트롤 플레인 IAM 역할은 필요한 권한이 있는 계정에 계속 존재합니다.
3.EKS 보안 그룹: 컨트롤 플레인 기본 보안 그룹은 필요한 액세스 규칙이 있는 계정에서 계속 사용할 수 있습니다.
4.클러스터에 비밀 암호화가 활성화되어 있는 경우 클러스터 IAM 역할에 AWS KMS (AWS 키 관리 서비스) 키를 사용할 권한이 있는지 확인하십시오.

### 사용 가능한 IP 주소 확인

클러스터를 업데이트하려면 Amazon EKS에서 클러스터를 생성할 때 지정한 서브넷의 사용 가능한 IP 주소가 최대 5개까지 필요합니다.

다음 명령을 실행하여 서브넷에 클러스터를 업그레이드할 수 있는 충분한 IP 주소가 있는지 확인할 수 있습니다.

```
CLUSTER=<cluster name>
aws ec2 describe-subnets --subnet-ids \
  $(aws eks describe-cluster --name ${CLUSTER} \
  --query 'cluster.resourcesVpcConfig.subnetIds' \
  --output text) \
  --query 'Subnets[*].[SubnetId,AvailabilityZone,AvailableIpAddressCount]' \
  --output table

----------------------------------------------------
|                  DescribeSubnets                 |
+---------------------------+--------------+-------+
|  subnet-067fa8ee8476abbd6 |  us-east-1a  |  8184 |
|  subnet-0056f7403b17d2b43 |  us-east-1b  |  8153 |
|  subnet-09586f8fb3addbc8c |  us-east-1a  |  8120 |
|  subnet-047f3d276a22c6bce |  us-east-1b  |  8184 |
+---------------------------+--------------+-------+
```

[VPC CNI 지표 도우미](https://github.com/aws/amazon-vpc-cni-k8s/blob/master/cmd/cni-metrics-helper/README.md) 를 사용하여 VPC 지표에 대한 CloudWatch 대시보드를 만들 수 있습니다. 

### EKS IAM 역할 확인

다음 명령을 실행하여 IAM 역할을 사용할 수 있고 계정에 올바른 역할 수임 정책이 적용되었는지 확인할 수 있습니다.

```
CLUSTER=<cluster name>
ROLE_ARN=$(aws eks describe-cluster --name ${CLUSTER} \
  --query 'cluster.roleArn' --output text)
aws iam get-role --role-name ${ROLE_ARN##*/} \
  --query 'Role.AssumeRolePolicyDocument'
  
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "eks.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

### EKS 보안 그룹 확인

계정에 보안 그룹이 있는지 확인하려면 다음 명령을 실행할 수 있습니다.

```
CLUSTER=<cluster name>
aws ec2 describe-security-groups \
  --group-ids $(aws eks describe-cluster \
    --query 'cluster.resourcesVpcConfig.securityGroupIds[*]' \
    --name ${CLUSTER} --output text)
    
{                                                                                                                                      
    "SecurityGroups": [                                                                                                                
        {                                                                                                                              
            "Description": "Communication between the control plane and worker nodegroups",                                            
            "GroupName": "eksctl-prefix-cluster-ControlPlaneSecurityGroup-HFO1JTSFWL1J",
```

다음과 같은 출력이 표시되면 클러스터에서 사용하고 계정에서 사용할 수 있는 보안 그룹을 다시 확인해야 합니다.

```
An error occurred (InvalidGroupId.Malformed) when calling the DescribeSecurityGroups operation: Invalid id:
```

보안 그룹을 찾을 수 없는 경우 [Amazon EKS 보안 그룹 요구 사항 및 고려 사항](https://docs.aws.amazon.com/eks/latest/userguide/sec-group-reqs.html) 을 검토하십시오.`eks-cluster-sg-my-cluster-uniqueID` 보안 그룹이 삭제된 경우 클러스터를 다시 생성해야 할 수 있습니다. 

## EKS 애드온으로 마이그레이션

Amazon EKS는 쿠버네티스용 Amazon VPC CNI 플러그인, `kube-proxy`, CoreDNS와 같은 애드온을 모든 클러스터에 자동으로 설치합니다.애드온은 자체 관리형이거나 Amazon EKS 애드온으로 설치할 수 있습니다.Amazon EKS 애드온은 EKS API를 사용하여 애드온을 관리하는 또 다른 방법입니다. 

Amazon EKS 애드온을 사용하여 단일 명령으로 버전을 업데이트할 수 있습니다.예를 들면 다음과 같습니다.

```
aws eks update-addon —cluster-name my-cluster —addon-name vpc-cni —addon-version version-number \
--service-account-role-arn arn:aws:iam::111122223333:role/role-name —configuration-values '{}' —resolve-conflicts PRESERVE
```

EKS 애드온을 사용한다면 다음의 명령어로 확인을 할 수 있습니다.

```
aws eks list-addons --cluster-name <cluster name>
```

!!! warning
      
    EKS Add-on은 컨트롤 플레인 업그레이드 중에 자동으로 업그레이드되지 않습니다.EKS 애드온 업데이트를 시작하고 원하는 버전을 선택해야 합니다. 

    * 사용 가능한 모든 버전 중에서 호환되는 버전을 선택하는 것은 사용자의 책임입니다.[애드온 버전 호환성 지침 검토](#쿠버네티스-API를-사용하여-애드온-및-컴포넌트-업그레이드) 
    * Amazon EKS 추가 기능은 한 번에 하나의 마이너 버전만 업그레이드할 수 있습니다. 

[EKS Add-on으로 사용할 수 있는 구성 요소 및 시작 방법에 대해 자세히 알아보십시오.](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html)

[EKS Add-on에 사용자 지정 구성을 제공하는 방법을 알아보십시오.](https://aws.amazon.com/blogs/containers/amazon-eks-add-ons-advanced-configuration/)

## 컨트롤 플레인을 업그레이드하기 전에 제거된 API 사용을 식별하고 수정하기

EKS 컨트롤 플레인을 업그레이드하기 전에 제거된 API의 API 사용을 확인해야 합니다. 이를 위해서는 실행 중인 클러스터 또는 정적으로 렌더링된 Kubernetes 매니페스트 파일을 확인할 수 있는 도구를 사용하는 것이 좋습니다. 

일반적으로 정적 매니페스트 파일을 대상으로 검사를 실행하는 것이 더 정확합니다. 라이브 클러스터를 대상으로 실행하면 이러한 도구가 오탐을 반환할 수 있습니다. 

쿠버네티스 API가 더 이상 사용되지 않는다고 해서 API가 제거된 것은 아닙니다. [쿠버네티스 지원 중단 정책](https://kubernetes.io/docs/reference/using-api/deprecation-policy/) 을 확인하여 API 제거가 워크로드에 미치는 영향을 이해해야 합니다.

### 큐브-노-트러블(Kube-no-Trouble)

[Kube-no-Trouble](https://github.com/doitintl/kube-no-trouble) 은 `kubent` 명령을 사용하는 오픈 소스 커맨드 라인 유틸리티입니다.인수 없이 `kubent'를 실행하면 현재 KubeConfig 컨텍스트를 사용하여 클러스터를 스캔하고 더 이상 사용되지 않고 제거될 API가 포함된 보고서를 인쇄합니다

```
kubent

4:17PM INF >>> Kube No Trouble `kubent` <<<
4:17PM INF version 0.7.0 (git sha d1bb4e5fd6550b533b2013671aa8419d923ee042)
4:17PM INF Initializing collectors and retrieving data
4:17PM INF Target K8s version is 1.24.8-eks-ffeb93d
4:l INF Retrieved 93 resources from collector name=Cluster
4:17PM INF Retrieved 16 resources from collector name="Helm v3"
4:17PM INF Loaded ruleset name=custom.rego.tmpl
4:17PM INF Loaded ruleset name=deprecated-1-16.rego
4:17PM INF Loaded ruleset name=deprecated-1-22.rego
4:17PM INF Loaded ruleset name=deprecated-1-25.rego
4:17PM INF Loaded ruleset name=deprecated-1-26.rego
4:17PM INF Loaded ruleset name=deprecated-future.rego
__________________________________________________________________________________________
>>> Deprecated APIs removed in 1.25 <<<
------------------------------------------------------------------------------------------
KIND                NAMESPACE     NAME             API_VERSION      REPLACE_WITH (SINCE)
PodSecurityPolicy   <undefined>   eks.privileged   policy/v1beta1   <removed> (1.21.0)
```

정적 매니페스트 파일 및 helm 패키지를 스캔하는 데에도 사용할 수 있습니다.매니페스트를 배포하기 전에 문제를 식별하려면 지속적 통합 (CI) 프로세스의 일부로 `kubent` 를 실행하는 것이 좋습니다.또한 매니페스트를 스캔하는 것이 라이브 클러스터를 스캔하는 것보다 더 정확합니다. 

Kube-no-Trouble은 클러스터를 스캔하기 위한 적절한 권한이 있는 샘플 [서비스 계정 및 역할](https://github.com/doitintl/kube-no-trouble/blob/master/docs/k8s-sa-and-role-example.yaml) 을 제공합니다. 

### 풀루토(pluto)

또 다른 옵션은 [pluto](https://pluto.docs.fairwinds.com/) 인데, 이는 라이브 클러스터, 매니페스트 파일, 헬름 차트 스캔을 지원하고 CI 프로세스에 포함할 수 있는 GitHub Action이 있다는 점에서 `kubent`와 비슷합니다.

```
pluto detect-all-in-cluster

NAME             KIND                VERSION          REPLACEMENT   REMOVED   DEPRECATED   REPL AVAIL  
eks.privileged   PodSecurityPolicy   policy/v1beta1                 false     true         true
```

## 쿠버네티스 워크로드 업데이트kubectl-convert를 사용하여 매니페스트를 업데이트.

업데이트가 필요한 워크로드와 매니페스트를 식별한 후에는 매니페스트 파일의 리소스 유형을 변경해야 할 수 있습니다 (예: 파드시큐리티폴리시를 파드시큐리티스탠다드로).이를 위해서는 리소스 사양을 업데이트하고 교체할 리소스에 따른 추가 조사가 필요합니다.

리소스 유형은 동일하게 유지되지만 API 버전을 업데이트해야 하는 경우, `kubectl-convert` 명령을 사용하여 매니페스트 파일을 자동으로 변환할 수 있습니다. 예전 디플로이먼트를 `apps/v1`으로 변환하는 경우를 예로 들 수 있다. 자세한 내용은 쿠버네티스 웹 사이트의 [kubectl convert 플러그인 설치](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin) 를 참조하십시오.

`kubectl-convert -f <file> --output-version <group>/<version>`

## 데이터 플레인이 업그레이드되는 동안 워크로드의 가용성을 보장하도록 PodDisruptionBudget 및 토폴로지 스프레드 제약 조건을 구성.

데이터 플레인이 업그레이드되는 동안 워크로드의 가용성을 보장하려면 워크로드에 적절한 [PodDisruptionBudget(파드디스럽션 예산)](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/#pod-disruption-budgets) 및 [토폴로지 스프레드 제약 조건] (https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints) 이 있는지 확인하십시오.모든 워크로드에 동일한 수준의 가용성이 필요한 것은 아니므로 워크로드의 규모와 요구 사항을 검증해야 합니다.

워크로드가 여러 가용 영역에 분산되어 있고 토폴로지가 분산된 여러 호스트에 분산되어 있는지 확인하면 워크로드가 문제 없이 새 데이터 플레인으로 자동 마이그레이션될 것이라는 신뢰도가 높아집니다. 

다음은 항상 80% 의 복제본을 사용할 수 있고 여러 영역과 호스트에 걸쳐 복제본을 분산시키는 워크로드의 예입니다.

```
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp
spec:
  minAvailable: "80%"
  selector:
    matchLabels:
      app: myapp
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 10
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - image: public.ecr.aws/eks-distro/kubernetes/pause:3.2
        name: myapp
        resources:
          requests:
            cpu: "1"
            memory: 256M
      topologySpreadConstraints:
      - labelSelector:
          matchLabels:
            app: host-zone-spread
        maxSkew: 2
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: DoNotSchedule
      - labelSelector:
          matchLabels:
            app: host-zone-spread
        maxSkew: 2
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
```

[AWS Resilience Hub](https://aws.amazon.com/resilience-hub/) 는 아마존 엘라스틱 쿠버네티스 서비스 (Amazon EKS) 를 지원 리소스로 추가했습니다. Resilience Hub는 애플리케이션 복원력을 정의, 검증 및 추적할 수 있는 단일 장소를 제공하므로 소프트웨어, 인프라 또는 운영 중단으로 인한 불필요한 다운타임을 피할 수 있습니다.

## 관리형 노드 그룹 또는 Karpenter를 사용하여 데이터 플레인 업그레이드를 간소화.

관리형 노드 그룹과 Karpenter는 모두 노드 업그레이드를 단순화하지만 접근 방식은 다릅니다.

관리형 노드 그룹은 노드의 프로비저닝 및 라이프사이클 관리를 자동화합니다.즉, 한 번의 작업으로 노드를 생성, 자동 업데이트 또는 종료할 수 있습니다.

기본 구성에서 Karpenter는 호환되는 최신 EKS 최적화 AMI를 사용하여 새 노드를 자동으로 생성합니다. EKS가 업데이트된 EKS 최적화 AMI를 출시하거나 클러스터가 업그레이드되면 Karpenter는 자동으로 이러한 이미지를 사용하기 시작합니다.[Karpenter는 노드 업데이트를 위한 노드 만료도 구현합니다.](#카펜터-관리형-노드의-노드-만료-활성화)
 
[Karpenter는 사용자 지정 AMI를 사용하도록 구성할 수 있습니다.](https://karpenter.sh/docs/concepts/node-templates/) Karpenter에서 사용자 지정 AMI를 사용하는 경우 kubelet 버전에 대한 책임은 사용자에게 있습니다. 

## 노드의 버전 스큐를 추적하여 업그레이드하기 전에 관리형 노드 그룹이 컨트롤 플레인과 동일한 버전에 있는지 확인

[업스트림 쿠버네티스 프로젝트](https://kubernetes.io/releases/version-skew-policy/) 는 컨트롤 플레인과 데이터 플레인 (특히 API 서버와 kubelet) 사이의 마이너 버전 스큐를 2개 마이너 버전으로 지원합니다.EKS 버전이 1.25인 경우 사용할 수 있는 가장 오래된 kubelet은 1.23입니다.

이 버전 차이는 EKS 관리형 노드 그룹과 EKS Fargate 프로파일로 생성한 노드마다 다릅니다.컨트롤 플레인과 데이터 플레인 간의 마이너 버전 스큐는 1개만 지원합니다 (예: EKS 1.25, 1.24 kubelet)

## 카펜터 관리형 노드의 노드 만료 활성화

Karpenter가 노드 업그레이드를 구현하는 한 가지 방법은 노드 만료라는 개념을 사용하는 것입니다.이렇게 하면 노드 업그레이드에 필요한 계획이 줄어듭니다.프로비저너에서 **ttlSecondsUntilExpired** 의 값을 설정하면 노드 만료가 활성화됩니다.노드가 몇 초 만에 정의된 연령에 도달하면 노드가 안전하게 비워지고 삭제됩니다.이는 사용 중인 경우에도 마찬가지이므로 노드를 새로 프로비저닝된 업그레이드된 인스턴스로 교체할 수 있습니다.노드가 교체되면 Karpenter는 EKS에 최적화된 최신 AMI를 사용합니다.자세한 내용은 카펜터 웹 사이트의 [프로비저닝 해제](https://karpenter.sh/docs/concepts/deprovisioning/#methods) 를 참조하십시오.

Karpenter는 이 값에 지터를 자동으로 추가하지 않습니다. 과도한 워크로드 중단을 방지하려면 Kubernetes 설명서에 나와 있는 대로 [파드 중단 예산](https://kubernetes.io/docs/tasks/run-application/configure-pdb/) 을 정의하십시오.

프로비저너에서 **ttlSecondsUntilExpired** 값을 설정하는 경우 이는 프로비저너와 연결된 기존 노드에 적용됩니다.

## 카펜터 관리 노드에 드리프트 기능 사용

[카펜터's Drift 기능](https://karpenter.sh/docs/concepts/deprovisioning/#drift) 은 Karpenter가 프로비저닝한 노드를 자동으로 업그레이드하여 EKS 컨트롤 플레인과 동기화 상태를 유지할 수 있습니다.카펜터 드리프트는 현재 [기능 게이트](https://karpenter.sh/docs/concepts/settings/#feature-gates) 를 사용하여 활성화해야 합니다.Karpenter의 기본 구성은 EKS 클러스터의 컨트롤 플레인과 동일한 메이저 및 마이너 버전에 대해 EKS에 최적화된 최신 AMI를 사용합니다.

EKS 클러스터 업그레이드가 완료되면 Karpenter의 Drift 기능은 Karpenter가 프로비저닝한 노드가 이전 클러스터 버전용 EKS 최적화 AMI를 사용하고 있음을 감지하고 해당 노드를 자동으로 연결, 드레인 및 교체합니다.새 노드로 이동하는 파드를 지원하려면 적절한 파드 [리소스 할당량](https://kubernetes.io/docs/concepts/policy/resource-quotas/) 을 설정하고 [파드 중단 예산](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) (PDB) 을 사용하여 쿠버네티스 모범 사례를 따르세요.Karpenter의 프로비저닝 해제는 파드 리소스 요청을 기반으로 대체 노드를 미리 가동하고 노드 프로비저닝을 취소할 때 PDB를 존중합니다.

## eksctl을 사용하여 자체 관리형 노드 그룹의 업그레이드를 자동화

자체 관리형 노드 그룹은 사용자 계정에 배포되고 EKS 서비스 외부의 클러스터에 연결된 EC2 인스턴스입니다.이들은 일반적으로 일종의 자동화 도구를 통해 배포되고 관리됩니다.자체 관리형 노드 그룹을 업그레이드하려면 도구 설명서를 참조해야 합니다.

예를 들어 eksctl은 [자체 관리형 노드 삭제 및 드레이닝](https://eksctl.io/usage/managing-nodegroups/#deleting-and-draining)을 지원합니다.

몇 가지 일반적인 도구는 다음과 같습니다.

* [eksctl](https://eksctl.io/usage/nodegroup-upgrade/)
* [kOps](https://kops.sigs.k8s.io/operations/updates_and_upgrades/)
* [EKS Blueprints](https://aws-ia.github.io/terraform-aws-eks-blueprints/node-groups/#self-managed-node-groups)

## 업그레이드 전 클러스터 백업

새 버전의 쿠버네티스는 Amazon EKS 클러스터를 크게 변경합니다.클러스터를 업그레이드한 후에는 다운그레이드할 수 없습니다.

[Velero](https://velero.io/) 는 커뮤니티에서 지원하는 오픈 소스 도구로, 기존 클러스터를 백업하고 새 클러스터에 백업을 적용하는 데 사용할 수 있습니다.

현재 EKS에서 지원하는 Kubernetes 버전에서만 새 클러스터를 생성할 수 있다는 점에 유의하십시오. 클러스터가 현재 실행 중인 버전이 계속 지원되고 업그레이드가 실패할 경우 원래 버전으로 새 클러스터를 생성하고 데이터 플레인을 복원할 수 있습니다. 참고로 IAM을 포함한 AWS 리소스는 Velero의 백업에 포함되지 않습니다. 이러한 리소스는 다시 생성해야 합니다. 

## 컨트롤 플레인을 업그레이드한 후 Fargate 배포를 다시 시작.

Fargate 데이터 플레인 노드를 업그레이드하려면 워크로드를 재배포해야 합니다. 모든 파드를 `-o wide` 옵션으로 나열하여 파게이트 노드에서 실행 중인 워크로드를 식별할 수 있습니다.'fargate-'로 시작하는 모든 노드 이름은 클러스터에 재배포해야 합니다.


## 인플레이스 클러스터 업그레이드의 대안으로 블루/그린 클러스터 평가

일부 고객은 블루/그린 업그레이드 전략을 선호합니다. 여기에는 이점이 있을 수 있지만 고려해야 할 단점도 있습니다.

혜택은 다음과 같습니다.

* 여러 EKS 버전을 한 번에 변경할 수 있습니다 (예: 1.23에서 1.25).
* 이전 클러스터로 다시 전환 가능
* 최신 시스템 (예: terraform) 으로 관리할 수 있는 새 클러스터를 생성합니다.
* 워크로드를 개별적으로 마이그레이션할 수 있습니다.

몇 가지 단점은 다음과 같습니다.

* 소비자 업데이트가 필요한 API 엔드포인트 및 OIDC 변경 (예: kubectl 및 CI/CD)
* 마이그레이션 중에 2개의 클러스터를 병렬로 실행해야 하므로 비용이 많이 들고 지역 용량이 제한될 수 있습니다.
* 워크로드가 서로 종속되어 함께 마이그레이션되는 경우 더 많은 조정이 필요합니다.
* 로드 밸런서와 외부 DNS는 여러 클러스터에 쉽게 분산될 수 없습니다.

이 전략은 가능하지만 인플레이스 업그레이드보다 비용이 많이 들고 조정 및 워크로드 마이그레이션에 더 많은 시간이 필요합니다.상황에 따라 필요할 수 있으므로 신중하게 계획해야 합니다.

높은 수준의 자동화와 GitOps와 같은 선언형 시스템을 사용하면 이 작업을 더 쉽게 수행할 수 있습니다.데이터를 백업하고 새 클러스터로 마이그레이션하려면 상태 저장 워크로드에 대한 추가 예방 조치를 취해야 합니다.

자세한 내용은 다음 블로그 게시물을 검토하세요.

* [쿠버네티스 클러스터 업그레이드: 블루-그린 배포 전략](https://aws.amazon.com/blogs/containers/kubernetes-cluster-upgrade-the-blue-green-deployment-strategy/)
* [스테이트리스 ArgoCD 워크로드를 위한 블루/그린 또는 카나리아 Amazon EKS 클러스터 마이그레이션](https://aws.amazon.com/blogs/containers/blue-green-or-canary-amazon-eks-clusters-migration-for-stateless-argocd-workloads/)

## Kubernetes 프로젝트에서 계획된 주요 변경 사항 추적 — 미리 생각해 보세요

다음 버전만 바라보지 마세요. Kubernetes의 새 버전이 출시되면 이를 검토하고 주요 변경 사항을 확인하십시오. 예를 들어, 일부 애플리케이션은 도커 API를 직접 사용했고, 도커용 컨테이너 런타임 인터페이스 (CRI) (Dockershim이라고도 함) 에 대한 지원은 쿠버네티스 `1.24`에서 제거되었습니다. 이런 종류의 변화에는 대비하는 데 더 많은 시간이 필요합니다. 
 
업그레이드하려는 버전에 대해 문서화된 모든 변경 사항을 검토하고 필요한 업그레이드 단계를 기록해 두십시오.또한 Amazon EKS 관리형 클러스터와 관련된 모든 요구 사항 또는 절차를 기록해 두십시오.

* [쿠버네티스 변경 로그](https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG)

## 기능 제거에 대한 구체적인 지침

### 1.25에서 Dockershim 제거 - Docker Socket(DDS) 용 검출기 사용

1.25용 EKS 최적화 AMI에는 더 이상 도커심에 대한 지원이 포함되지 않습니다.Dockershim에 대한 종속성이 있는 경우 (예: Docker 소켓을 마운트하는 경우) 작업자 노드를 1.25로 업그레이드하기 전에 이러한 종속성을 제거해야 합니다. 

1.25로 업그레이드하기 전에 Docker 소켓에 종속 관계가 있는 인스턴스를 찾아보세요.kubectl 플러그인인 [도커 소켓용 디텍터 (DDS)](https://github.com/aws-containers/kubectl-detector-for-docker-socket) 를 사용하는 것이 좋습니다. 

### Removal of PodSecurityPolicy in 1.25 - Migrate to Pod Security Standards or a policy-as-code solution

### 1.25에서 파드시큐리티폴리시 제거 - 파드 보안 표준 또는 코드형 정책 솔루션으로 마이그레이션

`PodSecurityPolicy` 는 [쿠버네티스 1.21에서 더 이상 사용되지 않음](https://kubernetes.io/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/)이 되었고, 쿠버네티스 1.25에서는 제거되었습니다.클러스터에서 PodSecurityPolicy를 사용하는 경우, 워크로드 중단을 방지하기 위해 클러스터를 버전 1.25로 업그레이드하기 전에 내장된 쿠버네티스 파드 보안 표준 (PSS) 또는 코드형 정책 솔루션으로 마이그레이션해야 합니다. 

AWS는 [EKS 설명서에 자세한 FAQ](https://docs.aws.amazon.com/eks/latest/userguide/pod-security-policy-removal-faq.html)를 게시했습니다.

[파드 보안 표준 (PSS) 및 파드 보안 승인 (PSA)](https://aws.github.io/aws-eks-best-practices/security/docs/pods/#pod-security-standards-pss-and-pod-security-admission-psa) 모범 사례를 검토하십시오. 

### 1.23에서 인트리 스토리지 드라이버 지원 중단 - 컨테이너 스토리지 인터페이스 (CSI) 드라이버로 마이그레이션

컨테이너 스토리지 인터페이스 (CSI) 는 Kubernetes가 기존의 인트리 스토리지 드라이버 메커니즘을 대체할 수 있도록 설계되었습니다.Amazon EBS 컨테이너 스토리지 인터페이스 (CSI) 마이그레이션 기능은 Amazon EKS `1.23` 이상 클러스터에서 기본적으로 활성화됩니다.파드가 버전 `1.22` 또는 이전 클러스터에서 실행되는 경우, 서비스 중단을 방지하려면 클러스터를 버전 `1.23`으로 업데이트하기 전에 [Amazon EBS CSI 드라이버](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html) 를 설치해야 합니다. 

[Amazon EBS CSI 마이그레이션 자주 묻는 질문](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi-migration-faq.html) 을 검토하십시오.

## 추가 리소스

### ClowdHaus EKS 업그레이드 지침

[ClowdHaus EKS 업그레이드 지침](https://clowdhaus.github.io/eksup/) 은 Amazon EKS 클러스터를 업그레이드하는 데 도움이 되는 CLI입니다.클러스터를 분석하여 업그레이드 전에 해결해야 할 잠재적 문제를 찾아낼 수 있습니다. 

### GoNoGo

[GoNoGo](https://github.com/FairwindsOps/GoNoGo) 는 클러스터 애드온의 업그레이드 신뢰도를 결정하는 알파 단계 도구입니다. 


