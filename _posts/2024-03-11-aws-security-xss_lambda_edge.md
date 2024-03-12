---
title: AWS Lambd@Edge를 활용하여 크로스 사이트 스크립팅(XSS) 방지하기
author: shlee
date: 2024-03-11 08:32:00 +0800
categories: [Blogging, AWS]
tags: [AWS, Security]
---

회사는 두 개의 출처에서 콘텐츠를 전달하기 위해 Amazon CloudFront 배포를 사용하고 있습니다. 하나의 출처는 Amazon EC2 인스턴스에 호스팅된 동적 애플리케이션이고, 다른 하나는 정적 자산을 위한 Amazon S3 버킷입니다. 보안 분석에 따르면, 애플리케이션에서 HTTPS 응답이 프레임 관련 크로스 사이트 스크립팅 공격을 방지하기 위해 X-Frame-Options HTTP 헤더를 제공하는 보안 요구사항을 준수하지 않는 것으로 나타났습니다.
보안 엔지니어는 누락된 HTTP 헤더를 응답에 추가함으로써 전체 스택을 준수하도록 해야 합니다. 이 요구사항을 충족하는 솔루션은 무엇인가요?

웹사이트를 방문할 때마다 브라우저는 웹 페이지를 요청하고, 서버는 HTTP 헤더와 함께 콘텐츠를 응답합니다. 캐시 제어와 같은 헤더는 브라우저가 콘텐츠를 얼마나 오래 캐시할지 결정하는 데 사용되며, 콘텐츠 유형과 같은 다른 헤더는 리소스의 미디어 유형을 나타내어 해당 리소스를 어떻게 해석할지를 지시합니다. 

애플리케이션 구성을 수정함으로써 보안 응답 헤더를 추가하는 것이 종종 가능합니다. 이 블로그에서는 원본에서 수정할 수 없는 애플리케이션(예: Amazon S3에 호스팅된 웹사이트)을 가지고 있을 때 동일한 결과를 어떻게 달성할 수 있는지에 초점을 맞추겠습니다.

## 크로스 사이트 스크립팅(XSS)이란 무엇인가요?

크로스 사이트 스크립팅은 웹사이트에 악성 스크립트를 주입하는 행위를 말합니다. 해커는 사람들이 친밀하고 안전하다고 생각하는 웹사이트에 악성 스크립트를 주입하고, 악성 스크립트가 포함된 게시글을 열람한 피해자들의 쿠키는 해커에게 전송됩니다. 이를 통해 해커는 피해자의 브라우저에서 스트립트를 실행해 사용자의 세션을 가로채거나, 웹사이트 변조하거나, 악의적인 컨텐츠 삽입하거나, 피싱 공격 등을 시도할 수 있게 됩니다.

![png](/assets/img/posts/aws_security_tip_xss_lamda_edge/1.png){:width="80%" height="80%"}

크로스 사이트 스크립팅은 스크립트 언어와 취약한 코드를 공격 대상으로 하며, 해킹의 주요 목적은 사용자의 정보를 도용하는 것이며, 로그인 입력란을 감염시켜 로그인 세부 정보와 쿠키를 탈취하는 방식으로 진행됩니다. 악성 소프트웨어는 사용자의 세부 정보를 기록해 해커에게 전송하고, 해커는 해당 정보를 사용해 피해자의 계정을 제어할 수 있게 됩니다.

## 보안 헤더란 무엇인가요?

보안 헤더는 서버에서 HTTP 응답에 포함된 헤더 그룹으로, 브라우저에게 사이트 콘텐츠를 처리할 때 어떻게 행동해야 하는지 알려줍니다. 예를 들어, X-XSS-Protection은 Internet Explorer와 Chrome이 크로스 사이트 스크립팅(XSS) 공격을 감지할 때 페이지 로딩을 중지하도록 하는 헤더입니다. 다음은 구현할 각 헤더의 목록과 추가 정보에 대한 링크입니다.

- Strict Transport Security
- Content-Security-Policy
- X-Content-Type-Options
- X-Frame-Options
- X-XSS-Protection
- Referrer-Policy

## Lambda@Edge 개요

Lambda@Edge는 Amazon CloudFront 엣지 위치에서 Lambda 함수를 실행할 수 있는 기능을 제공합니다. 이 기능은 고객에게 가까운 위치(지연 시간 측면에서)에서 HTTP 요청을 지능적으로 처리할 수 있도록 합니다. 시작하기 위해서는 코드(Lambda 함수를 Node.js로 작성)를 업로드하기만 하면 되며, 배포와 연관된 CloudFront 동작 중 하나를 선택하면 됩니다.

Lambda@Edge 함수는 CloudFront 이벤트 4가지에 대응하여 실행될 수 있습니다. 이 블로그 포스트의 목적을 위해, 우리는 Origin Response 이벤트에만 집중할 것입니다. 

Origin Response – 이 이벤트는 원본이 요청에 대한 응답을 반환한 후에 트리거됩니다. 원본으로부터의 응답에 접근할 수 있습니다.

![png](/assets/img/posts/aws_security_tip_xss_lamda_edge/2.png)

다음 다이어그램은 CloudFront 배포에 사용할 수 있는 트리거를 나타냅니다. 우리는 번호 6에 집중하고 있습니다.
1. 방문자가 웹사이트로 이동합니다.
2. CloudFront가 캐시에서 콘텐츠를 제공하기 전에, 해당 동작에 관련된 Viewer Request 트리거를 발동시키는 람다 함수를 실행합니다.
3. CloudFront가 캐시에서 콘텐츠를 제공합니다. 이용 가능한 경우, 그렇지 않으면 4단계로 이동합니다.
4. CloudFront 캐시 ‘Miss’ 후에만, 해당 동작에 대한 Origin Request 트리거가 발동됩니다.
5. S3 Origin이 콘텐츠를 반환합니다.
6. S3에서 콘텐츠가 반환된 후, 하지만 CloudFront에 캐싱되기 전에, Origin Response 트리거가 발동됩니다.
7. 콘텐츠가 CloudFront에 캐싱된 후, Viewer Response 트리거가 발동되며 이는 방문자가 콘텐츠를 받기 전 마지막 단계입니다.
8. Viewer가 콘텐츠를 받습니다.

## 솔루션 개요

이 솔루션은 Amazon S3 버킷에 호스팅된 간단한 단일 페이지 웹사이트와 Amazon CloudFront를 사용합니다. CloudFront 배포와 연동하는 새로운 Lambda@Edge 함수를 생성하는 방법, 그리고 Amazon CloudWatch 로그를 사용하여 실행을 모니터링하는 방법을 보여드리겠습니다. 원점 응답 트리거를 사용하여 Lambda@Edge 함수를 실행할 것입니다. 이를 통해 CloudFront가 보안 헤더가 추가된 응답을 캐시할 수 있게 되며, 이는 Lambda@Edge 함수가 CloudFront ‘Miss’ 시에만 트리거되어야 하며, 모든 미래의 ‘Hits’에 대해 보안 헤더가 반환된다는 의미입니다.

웹사이트용 S3 버킷이 이미 구성되어 있으며, 콘텐츠 제공을 위해 CloudFront 배포가 구성되어 있다고 가정하겠습니다. 데모 목적으로, 저는 S3 버킷을 설정하고, 배포의 원점으로 사용하며, “Hello World! Do I have security headers yet?”라는 텍스트가 있는 기본 index.html 파일을 업로드했습니다.

다음 다이어그램은 Lambda@Edge 함수를 트리거하는 이벤트 순서를 보여줍니다:

다음은 프로세스가 작동하는 방식입니다:

![png](/assets/img/posts/aws_security_tip_xss_lamda_edge/3.png)

1. 방문자가 www.example.com 웹사이트를 요청합니다.
2. 객체가 이미 캐시되어 있다면, CloudFront는 캐시에서 객체를 방문자에게 반환합니다. 그렇지 않으면 3단계로 진행됩니다.
3. CloudFront가 원본, 이 경우 S3 버킷에서 객체를 요청합니다.
4. S3가 객체를 반환하며, 이는 차례로 CloudFront가 원본 응답 이벤트를 트리거하게 합니다.
5. 우리의 Add Security Headers Lambda 함수가 트리거되고, 그 결과 출력은 CloudFront에 의해 캐시되어 제공됩니다.

이 코드는 응답의 내용을 가져오고, 새 헤더를 설정한 다음, 새 보안 헤더를 포함한 업데이트된 응답을 반환합니다.
![png](/assets/img/posts/aws_security_tip_xss_lamda_edge/4.png){:width="80%" height="80%"}

```javascript
'use strict';
exports.handler = (event, context, callback) => {
    
    //Get contents of response
    const response = event.Records[0].cf.response;
    const headers = response.headers;

//Set new headers 
    headers['strict-transport-security'] = [{key: 'Strict-Transport-Security', value: 'max-age=63072000; includeSubdomains; preload'}]; 
    headers['content-security-policy'] = [{key: 'Content-Security-Policy', value: "default-src 'none'; img-src 'self'; script-src 'self'; style-src 'self'; object-src 'none'"}]; 
    headers['x-content-type-options'] = [{key: 'X-Content-Type-Options', value: 'nosniff'}]; 
    headers['x-frame-options'] = [{key: 'X-Frame-Options', value: 'DENY'}]; 
    headers['x-xss-protection'] = [{key: 'X-XSS-Protection', value: '1; mode=block'}]; 
    headers['referrer-policy'] = [{key: 'Referrer-Policy', value: 'same-origin'}]; 

    //Return modified response
    callback(null, response);
};
```

## 결론
이 게시물에서는 CloudFront 배포 동작의 원본 응답 트리거에 보안 헤더를 추가하여 웹사이트의 보안을 개선하기 위해 Lambda@Edge를 사용하는 방법을 보여주었습니다. Lambda@Edge에 대한 활용성에 대해서 다시 한번 생각해볼수 있는 계기가 되었던 것 같네요.