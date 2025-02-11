<!-- # Container image scanning -->
# 컨테이너 이미지 스캔

<!-- Image Scanning is an automated vulnerability assessment feature that helps improve the security of your application’s container images by scanning them for a broad range of operating system vulnerabilities. -->
이미지 스캔은 광범위한 운영체제 취약성을 검사하여 애플리케이션 컨테이너 이미지의 보안을 개선하는 데 도움이 되는 자동화된 취약성 평가 기능입니다.

<!-- Currently, the Amazon Elastic Container Registry (ECR) is only able to scan Linux container image for vulnerabilities. However; there are third-party tools which can be integrated with an existing CI/CD pipeline for Windows container image scanning. -->
현재 Amazon Elastic Container Registry(ECR)는 Linux 컨테이너 이미지만 스캔하여 취약점을 찾아낼 수 있습니다.하지만 Windows 컨테이너 이미지 스캔을 위해 기존 CI/CD 파이프라인과 통합할 수 있는 타사 도구가 있습니다.

* [Anchore](https://anchore.com/blog/scanning-windows-container-images/)
* [PaloAlto Prisma Cloud ](https://docs.paloaltonetworks.com/prisma/prisma-cloud/prisma-cloud-admin-compute/vulnerability_management/windows_image_scanning.html)
* [Trend Micro - Deep Security Smart Check](https://www.trendmicro.com/en_us/business/products/hybrid-cloud/smart-check-image-scanning.html)

<!-- To learn more about how to integrate these solutions with Amazon Elastic Container Repository (ECR), check: -->
이러한 솔루션을 Amazon Elastic Container Registry(ECR) 와 통합하는 방법에 대해 자세히 알아보려면 다음을 확인하십시오:

* [Anchore, Amazon Elastic Container Registry (ECR)에서 이미지 스캔](https://anchore.com/blog/scanning-images-on-amazon-elastic-container-registry/)
* [PaloAlto, Amazon Elastic Container Registry (ECR)에서 이미지 스캔](https://docs.paloaltonetworks.com/prisma/prisma-cloud/prisma-cloud-admin-compute/vulnerability_management/registry_scanning0/scan_ecr.html)
* [TrendMicro, Amazon Elastic Container Registry (ECR)에서 이미지 스캔](https://cloudone.trendmicro.com/docs/container-security/sc-about/)
