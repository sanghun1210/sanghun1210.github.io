---
title: AWS WAF와 Network Firewall의 차이
author: shlee
date: 2024-02-21 00:34:00 +0800
categories: [Blogging, Tutorial]
tags: [aws]
---

AWS WAF (웹 애플리케이션 방화벽)과 AWS Network Firewall은 모두 악의적인 트래픽으로부터 AWS 리소스를 보호하는 네트워크 보안 서비스입니다. 그러나 각각의 장단점이 있으며, 사용 사례에 따라 가장 적합한 서비스가 다릅니다. 다음 다이어그램은 AWS 문서에서 가져왔습니다.

## AWS WAF:
![작동 방식](/assets/img/aws/WAF_Flowchart.png)
AWS WAF는 클라우드 기반 웹 애플리케이션 방화벽으로 SQL 인젝션 및 크로스 사이트 스크립팅과 같은 일반적인 웹 공격으로부터 보호합니다. OSI 모델의 애플리케이션 계층(7계층)에서 작동하여 HTTP 및 HTTPS 트래픽을 검사하고 악의적인 요청을 차단할 수 있습니다. AWS WAF는 인터넷에 노출된 웹 애플리케이션을 보호하기에 좋은 선택입니다.

AWS WAF를 사용하면 가용성에 영향을 줄 수 있는 일반적인 웹 공격 및 봇을 방지할 수 있습니다.

## AWS Network Firewall:
![작동 방식](/assets/img/aws/NFW_Flowchart.png)
AWS Network Firewall은 웹 트래픽, 데이터베이스 트래픽, 스트리밍 트래픽 등 모든 종류의 네트워크 트래픽을 보호하는 관리형, 상태 유지 방화벽입니다. OSI 모델의 네트워크 계층(3계층)에서 작동하여 IP 패킷을 검사하고 출발지 및 목적지 IP 주소, 포트, 프로토콜을 기반으로 악의적인 트래픽을 차단할 수 있습니다. AWS Network Firewall은 웹 애플리케이션, 데이터베이스, 서버를 포함한 다양한 AWS 리소스를 보호하기에 좋은 선택입니다.

AWS Network Firewall을 사용하면 네트워크 트래픽에 대한 세밀한 제어를 제공하는 방화벽 규칙을 정의할 수 있습니다. Network Firewall은 AWS Firewall Manager와 함께 작동하여 Network Firewall 규칙을 기반으로 정책을 구축하고 해당 정책을 가상 사설 클라우드(VPC) 및 계정에 중앙 집중식으로 적용할 수 있습니다.

다음은 AWS WAF와 AWS Network Firewall의 주요 차이점을 요약한 표입니다:
![작동 방식](/assets/img/aws/diff_aws_nfw_sheet.png)
|Feature|AWS WAF|AWS Network Firewall|
|--------------|---|---|
|Layer of protection|Application layer (layer7)|Network layer (layer 3-7)|
|Protection|Endpoint Level (ALB, ApiGateway, Cloudfront 등)|VPC Level|
|보호되는 트래픽 유형|Web traffic|All types of network traffic|
|Managed service|Yes|Yes|
|Stateful firewall|No|Yes|
|침임 탐지|No|Yes|
|비용|Pay-per-use|Pay-per-rule|
|State|Stateless|Statefule and stateless|
|Flows|Ingress from internet to Endpoint level|Ingress and Egress at VPC level|
|Features|Application layer filtering|Stateless/ACL L3 rules, Statefule/L4 rules, IPS-IDS/L7 rules, FQDN Filtering, Protocall detection, Ip Allow/Block lists|

일반적으로, AWS WAF는 웹 애플리케이션을 보호하기에 좋은 선택이며, AWS Network Firewall은 더 넓은 범위의 AWS 리소스를 보호하기에 좋은 선택입니다. 그러나 모든 상황에 완벽하게 맞는 솔루션은 없으며, 최적의 선택은 특정 요구 사항에 따라 달라질 것입니다.

## AWS WAF와 AWS Network Firewall 중에서 선택할 때 고려해야 할 몇 가지 추가 사항은 다음과 같습니다:

1. 복잡성: AWS WAF는 AWS Network Firewall보다 더 복잡한 서비스입니다. 더 많은 구성과 유지 관리가 필요하며, 문제 해결이 더 어려울 수 있습니다.
2. 비용: AWS WAF는 AWS Network Firewall보다 더 비쌉니다. 생성하는 규칙마다 비용을 지불해야 하며, 규칙이 검사하는 트래픽 양에 따라 비용도 지불해야 합니다.
3. 성능: AWS WAF는 웹 애플리케이션의 성능에 부정적인 영향을 미칠 수 있습니다. 이는 AWS WAF가 애플리케이션으로 전송되는 모든 HTTP 및 HTTPS 요청을 검사하기 때문입니다.
4. 규칙: AWS WAF는 허용되거나 거부되는 트래픽을 정의하는 데 규칙을 사용합니다. 이러한 규칙은 출발지 IP 주소, 목적지 IP 주소, HTTP 메서드, URL 경로와 같은 다양한 기준을 기반으로 할 수 있습니다. AWS Network Firewall은 허용되거나 거부되는 트래픽을 정의하는 데 규칙 그룹을 사용합니다. 이러한 규칙 그룹은 출발지 및 목적지 IP 주소, 포트, 프로토콜과 같은 다양한 기준을 기반으로 할 수 있습니다.
5. 확장성: AWS WAF는 트래픽 증가와 함께 확장될 수 있도록 설계되었습니다. 트래픽이 증가함에 따라 AWS WAF는 자동으로 처리량을 늘릴 수 있습니다. AWS Network Firewall 역시 트래픽 증가와 함께 확장될 수 있도록 설계되었지만, 다른 방식으로 이루어집니다. AWS Network Firewall은 분산 아키텍처를 사용하여 트래픽을 여러 서버에 분산시킬 수 있습니다. 이는 성능과 신뢰성을 향상시키는 데 도움이 됩니다.
6. 배포: AWS WAF는 다양한 방식으로 배포할 수 있습니다. 온프레미스, 가상 사설 클라우드(VPC), 하이브리드 환경에서 배포할 수 있습니다. AWS Network Firewall은 VPC에서만 사용할 수 있습니다.

## 결론:

AWS WAF는 악의적인 트래픽으로부터 웹 애플리케이션을 보호하는 데 도움이 되는 강력한 도구입니다. 다양한 AWS 서비스에 배포할 수 있으며 특정 요구 사항에 맞게 사용자 정의할 수 있는 다재다능한 서비스입니다. 웹 애플리케이션을 보호하는 방법을 찾고 있다면 AWS WAF는 고려해 볼 좋은 옵션입니다.
