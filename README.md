# Chapter 4-1. 성능 최적화 Part 1
<br> 

### 목차
1. [배포 파이프라인](#1-배포-파이프라인)
2. [GitHub Action](#2-github-action)
3. [주요 링크](#3-주요-링크)
4. [주요 개념](#4-주요-개념)
    - [GitHub Actions과 CI/CD 도구](#41-github-actions과-cicd-도구)
    - [S3와 스토리지](#42-s3와-스토리지)
    - [CloudFront와 CDN](#43-cloudfront와-cdn)
    - [캐시 무효화(Cache Invalidation)](#44-캐시-무효화cache-invalidation)
    - [Repository secret과 환경변수](#45-repository-secret과-환경변수)
5. [CDN 도입 여부에 따른 성능 개선 보고서](#5-cdn-도입-여부에-따른-성능-개선-보고서)
    - [테스트 환경](#51-테스트-환경)
    - [성능 분석](#52-성능-분석)
        - [동영상을 포함한 페이지 성능 분석](#521-동영상을-포함한-페이지-성능-분석)
        - [폰트를 포함한 페이지 성능 분석](#522-폰트를-포함한-페이지-성능-분석)
6. [요약 및 결론](#6-요약-및-결론)

<br> 

## 1. 배포 파이프라인

### 개요
![배포 파이프라인](https://github.com/user-attachments/assets/b992b2bd-f4c1-4d4c-86b5-6c119be70f06)

1. 개발자는 개발이 완료된 코드를 `main` 브랜치에 푸시합니다.
2. GitHub Actions을 통해 Next.js의 빌드 산출물(`.output`)이 S3 버킷으로 업로드됩니다.
3. 업로드된 정적 산출물은 CloudFront를 통해 CDN으로 사용자에게 제공됩니다.

<br>

## 2. GitHub Action

### 동작 과정
1. `main` 브랜치에 코드가 푸시되면 트리거됩니다.
2. 저장소를 체크아웃합니다.
3. Node.js를 설치합니다.
4. 프로젝트 의존성을 설치합니다.
5. 프로젝트를 빌드합니다.
6. AWS 자격증명을 구성합니다.
7. 빌드된 정적 파일을 S3 버킷에 동기화합니다.
8. CloudFront 캐시를 무효화합니다.


<br>

## 3. 주요 링크

- **S3 버킷 웹사이트 엔드포인트:**  
  [http://hanghae-project.s3-website.ap-northeast-2.amazonaws.com/](http://hanghae-project.s3-website.ap-northeast-2.amazonaws.com/)
- **CloudFront 배포 도메인 이름:**  
  [http://d1ujuut01ecq75.cloudfront.net](http://d1ujuut01ecq75.cloudfront.net)


<br>

## 4. 주요 개념

### 4.1. GitHub Actions과 CI/CD 도구
- **CI/CD 도구:** GitHub Actions, Jenkins, CircleCI 등
- **GitHub Actions:** GitHub와 통합되어 러닝 커브가 낮으며, 커밋, 풀 리퀘스트, 머지, 태그 생성 등의 이벤트를 기반으로 동작합니다.  
  코드를 테스트하고 빌드하며, 배포 프로세스를 자동화합니다.

### 4.2. S3와 스토리지
- **스토리지:** 데이터를 저장하고 관리하기 위한 시스템 또는 공간.
- **S3:** 객체 스토리지 서비스로, 정적 웹 호스팅을 지원하며, IAM 정책, 버킷 정책 등을 통해 권한 관리를 할 수 있습니다.

### 4.3. CloudFront와 CDN
- **CDN(Content Delivery Network):** 전 세계에 분산된 서버 네트워크를 통해 사용자에게 콘텐츠를 빠르고 안정적으로 전달합니다.
- **CloudFront:** 글로벌 CDN 서비스로, 엣지 서버를 통해 캐싱을 제공하며, S3와 EC2 등 AWS 서비스와 통합됩니다.

### 4.4. 캐시 무효화(Cache Invalidation)
- 캐시 무효화는 아래 시점에 동작합니다:
    - 명시적으로 캐시 무효화 요청(`aws cloudfront create-invalidation`)
    - 캐시 만료 시간(기본 24시간) 도달
    - 파일 변경 시 ETag 또는 Last-Modified 헤더 변경
    - CloudFront 배포 업데이트 시

### 4.5. Repository secret과 환경변수
- **환경변수:** 실행 환경에서 애플리케이션이 사용하는 값. 설정값, 경로 정보 등을 저장합니다.
- **Repository secret:** GitHub Actions에서 민감한 데이터를 보호하기 위한 보안 환경 변수. 주로 API 키, 클라우드 자격증명을 저장합니다.


<br>

## 5. CDN 도입 여부에 따른 성능 개선 보고서

### 5.1. 테스트 환경
- **대상:** 동영상을 포함한 페이지, 폰트를 포함한 페이지, 기본 Next.js 프로젝트
- **비교 배포 방식:**
    - **S3 단독 배포:** AWS S3를 정적 파일 저장소로 사용
    - **CloudFront 결합 배포:** AWS S3와 CloudFront(CDN)를 함께 사용
- **측정 도구:** 크롬 개발자 도구의 네트워크 탭

### 5.2 성능 분석
#### 5.2.1 동영상을 포함한 페이지 성능 분석
아래 이미지는 S3 단독 배포(왼쪽)와 CloudFront 결합 배포(오른쪽)의 네트워크 요청 비교입니다.

<img width="1290" alt="387479886-645ce111-3333-407d-b68e-379f0a468ba3" src="https://github.com/user-attachments/assets/a1cba794-0f9e-44d4-8060-7b34b74dd1ee">
<br>

| 항목                     | S3 배포 결과       | CloudFront 배포 결과 | 차이점 및 분석                               |
|--------------------------|--------------------|-----------------------|---------------------------------------------|
| **네트워크 요청 수**     | 30건              | 27건                 | CloudFront가 중복된 요청 수를 줄임.          |
| **로드 타임**            | 6.36초            | 6.04초               | 약 5% 더 빠른 로드 타임 제공.                |
| **비디오 요청 시간**     | 280ms (`mangom.mp4`) | 56ms (`mangom.mp4`) | 캐싱을 통해 약 80% 시간 단축.               |



#### 5.2.2 폰트를 포함한 페이지 성능 분석
아래 이미지는 S3 단독 배포(왼쪽)와 CloudFront 결합 배포(오른쪽)의 네트워크 요청(글꼴) 비교입니다.

<img width="1183" alt="388030105-d79b5600-7b19-4970-88b5-af314acf3f9c" src="https://github.com/user-attachments/assets/d048cd73-214e-4e05-bbcd-89eed53b1591">

<br>

| 항목                     | S3 배포 결과       | CloudFront 배포 결과 | 차이점 및 분석                               |
|--------------------------|--------------------|-----------------------|---------------------------------------------|
| **요청 수**              | 3건               | 3건                  | 동일.                                        |
| **DOMContentLoaded 시간**| 304ms             | 196ms                | CloudFront가 약 35% 더 빠른 DOM 로드 제공.   |
| **전송된 데이터 크기**    | 15.7KB            | 13.4KB               | 약 15% 데이터 전송량 감소.                   |


<br>

## 6. 요약 및 결론

### 요약
- **CDN 도입의 필요성:** 글로벌 사용자나 정적 리소스 사용량이 많은 경우 필수적입니다.
- **캐시 전략 최적화:** Cache-Control 헤더를 사용해 사용자 정의 캐싱 정책을 설정하세요.
- **비용 관리:** 트래픽이 많은 정적 리소스에 우선적으로 CDN을 활용하고, 덜 중요한 리소스는 S3만 사용하여 하이브리드 전략을 채택할 수 있습니다.

### 결론
CDN 도입은 S3 단독 배포에 비해 네트워크 요청 최적화, 캐싱 효율 증가, 페이지 로드 시간 개선, 서버 부하 감소 등의 이점을 제공합니다.
