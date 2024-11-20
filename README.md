## 목차
- [배포 파이프라인](#배포-파이프라인)
- [github action](#github-action)
- [주요 링크](#주요-링크)
- [주요 개념](#주요-개념)
- [CDN과 성능최적화/성능개선 보고서](#cdn과-성능최적화)

## 배포 파이프라인

### 개요
![hanghae-plus (4)](https://github.com/user-attachments/assets/b992b2bd-f4c1-4d4c-86b5-6c119be70f06)
1. 개발자는 개발이 완료된 코드를 main 브랜치에 푸시합니다.
2. github actions을 통해 Next.js의 빌드 산출물(.output)이 S3 버킷으로 업로드됩니다.
3. 업로드된 정적 산출물은 CloudFront를 통해 CDN으로 사용자에게 제공됩니다. 

## github action
1. main 브랜치에 코드가 푸시되면 트리거됩니다.
2. 저장소를 체크아웃합니다.
3. Node.js를 설치합니다.
4. 프로젝트 의존성을 설치합니다.
5. 프로젝트를 빌드합니다.
6. AWS 자격증명을 구성합니다.
7. 빌드된 정적 파일을 S3버킷에 동기화합니다.
8. CloudFront 캐시를 무효화합니다.

## 주요 링크

- S3 버킷 웹사이트 엔드포인트: http://hanghae-project.s3-website.ap-northeast-2.amazonaws.com/
- CloudFront 배포 도메인 이름: http://d1ujuut01ecq75.cloudfront.net

## 주요 개념

- GitHub Actions과 CI/CD 도구
  - CI/CD 도구 : github actions, jenkins, CircleCI 등
  - Github actions : github와 통합되어 있고 러닝커브가 낮은 편이며, 코드를 테스트하고 빌드하며, 배포 프로세스를 자동화 한다. 또한, 커밋, 풀리퀘스트, 머지, 태그 생성 등의 이벤트를 기반으로 동작한다.
- S3와 스토리지
  - 스토리지 : 데이터를 저장하고 관리하기 위한 시스템 또는 공간. 우리가 사용하는 S3는 객체 스토리지에 해당하며, 데이터를 객체 형태(파일 + 메타 데이터)로 저장한다. 
  - S3 : 객체 스토리지 서비스. 정적 웹 호스팅을 지원하며, IAM 정책, 버킷 정책 등을 통해 권한관리를 할 수 있고, 파일 변경 이력을 저장해 삭제나 수정된 파일을 복구할 수 있다.
- CloudFront와 CDN
  - CDN : 전 세계에 분산된 서버 네트워크를 통해 사용자에게 컨텐츠(이미지, 동영상, 정적 파일 등)를 빠르고 안정적이게 전달하는 기술. 사용자와 가까운 서버(엣지 서버)에 컨텐츠를 캐싱하여 지연시간을 줄이며, 서버 부하를 분산시켜 대규모 트래픽을 효율적으로 처리한다.
  - CloudFront : 글로벌 CDN 서비스. 정적 컨텐츠들을 엣지 서버에 케싱하며 S3, EC2, Lambda와 같은 AWS 서비스와 통합하여 컨텐츠를 빠르게 배포할 수 있다. HTTPS를 기본적으로 지원하고 캐시 불가능한 동적 컨텐츠(사용자별로 다른 데이터)도 최소한의 지연시간으로 제공한다.
- 캐시 무효화(Cache Invalidation)
  - 캐시 무효화는 어떤 시점에 동작하는가?
    - 명시적으로 캐시 무효화를 요청한 경우(명령어 ```aws cloudfront create-invalidation```)
    - 캐시 만료 시간에 따라 자동으로 동작(기본적으로 24시간으로 설정)
    - 파일변경으로 인해 ETag 또는 Last-Modified 헤더가 달라질때
    - CloudFront 배포 업데이트 시점
- Repository secret과 환경변수
  - 환경변수 : 실행환경에서 어플리케이션에서 전달되는 값으로 설정값, 경로 정보, 변수 등을 저장하는데 사용된다.
  - Repository secret : 민감한 데이터를 보호하기 위해 github actions와 같은 CI/CD도구에서 사용하는 보안 환경 변수이다. 주로 API 키, 클라우드 자격증명 등 민감한 정보를 저장한다.

  
## CDN 도입여부에 따른 성능개선 보고서
### 목적
CDN(Content Delivery Network)의 도입이 웹 애플리케이션 성능에 미치는 영향을 분석하고, S3 단독 배포와 CloudFront를 결합한 배포 방식의 성능 차이를 비교합니다.

### 테스트 환경
- 대상 : 동영상을 포함한 페이지, 기본 Next.js 프로젝트
- 비교 배포 방식:
  - S3 단독 배포: AWS S3를 정적 파일 저장소로 사용합니다.
  - CloudFront 결합 배포: AWS S3와 CloudFront(CDN)를 함께 사용합니다.
- 측정 도구: 크롬 개발자 도구의 네트워크 탭을 활용하여 요청, 로드 타임, 캐싱 효과를 분석합니다.

### 동영상을 포함한 페이지 성능 분석

#### 네트워크 탭 결과
아래 이미지는 S3 단독 배포(왼쪽)와 CloudFront 결합 배포(오른쪽)의 네트워크 요청 비교입니다.

<img width="1290" alt="image" src="https://github.com/user-attachments/assets/645ce111-3333-407d-b68e-379f0a468ba3">

#### 성능 비교

| 항목                     | S3 배포 결과             | CloudFront 배포 결과   | 차이점 및 분석                           |
|--------------------------|--------------------------|-------------------------|------------------------------------------|
| **네트워크 요청 수**     | 30건                    | 27건                   | CloudFront가 요청을 최적화하여 중복된 요청 수를 줄임. |
| **로드 타임 (Load Time)**| 6.36초                  | 6.04초                 | CloudFront가 약 **5% 빠른 로드 타임**을 제공. |
| **DOMContentLoaded 시간**| 0.51초                  | 0.57초                 | CloudFront는 초기 캐싱 처리로 약간의 지연 발생. |
| **비디오 파일 요청 시간**| 280ms (`mangom.mp4`)    | 56ms (`mangom.mp4`)    | CloudFront의 캐싱이 **80% 이상의 시간 단축**을 실현. |
| **리소스 캐싱**          | 서버에서 직접 응답       | 대부분 `304 Not Modified` 응답 | CloudFront가 **엣지 서버 캐싱**을 통해 효율적인 응답 제공. |
| **전송된 데이터 크기**   | 1.5MB                  | 1.47MB                | CloudFront는 압축 및 캐싱 최적화로 약간의 데이터 절감. |

### 추가 분석
- **비디오 파일 처리 성능**: 비디오 요청 시간이 CloudFront에서 280ms에서 56ms로 줄어들며, 엣지 서버의 캐싱 덕분에 큰 성능 향상을 보입니다.
- **로드 타임 개선**: CloudFront는 네트워크 요청 최적화 및 캐싱을 통해 S3 대비 약 5%의 로드 타임 개선을 제공합니다.
- **리소스 최적화**: CloudFront는 중복 요청을 줄이고, 캐싱된 리소스(`304 Not Modified`)를 적극적으로 활용하여 서버 부하를 감소시킵니다.


### 기본 Next.js 프로젝트 성능 분석

#### 네트워크 탭 결과
아래 이미지는 S3 단독 배포(왼쪽)와 CloudFront 결합 배포(오른쪽)의 네트워크 요청 비교입니다.

<img width="1108" alt="image" src="https://github.com/user-attachments/assets/484e731e-0e9d-409b-b3ec-05e2f551965a">

#### 성능 비교

| 측정 지표                | S3 결과    | CloudFront 결과 | 개선율           |
|--------------------------|------------|------------------|------------------|
| **페이지 로드 시간**      | 3초        | 1초              | **67% 개선**      |
| **TTFB(Time to First Byte)** | 1.5초      | 0.4초            | **73% 개선**      |

#### 추가 분석
- **페이지 로드 시간**: CloudFront 결합 배포는 S3 단독 배포 대비 **67% 빠른 로드 시간**을 보임. 이는 전 세계적으로 분산된 엣지 서버를 통해 사용자 근처에서 리소스를 제공한 결과입니다.
- **TTFB(Time to First Byte)**: 첫 바이트 응답 시간이 1.5초에서 0.4초로 **73% 단축**되어, 초기 페이지 로딩이 더욱 빨라집니다.
- **최적화 효과**: 캐싱된 리소스를 활용하고, 글로벌 분산 서버로 네트워크 지연을 최소화하여 사용자 경험이 향상시킵니다.

### 요약 및 결론

#### 성능 개선 요약
- CDN 도입을 통해 전반적인 성능이 크게 개선됩니다.
- 페이지 로드 시간과 TTFB(Time to First Byte) 모두 CloudFront 결합 배포에서 S3 단독 배포 대비 현저히 빨라집니다.
- 동영상 로딩 및 정적 리소스 요청에 대한 캐싱 효율이 증가하여 서버 부하가 감소합니다.

### 결론
1. **CDN 도입 권장**: 글로벌 사용자를 대상으로 하거나 정적 리소스(동영상, 이미지 등)의 사용량이 많은 웹 애플리케이션에는 CDN 도입이 필수적입니다.
2. **Cache-Control 헤더 설정**: CloudFront의 캐싱 정책을 사용자 정의하여 성능 최적화 및 비용 절감 효과를 극대화합니다.
3. **비용 관리**: CloudFront는 사용량에 따라 비용이 증가할 수 있으므로 트래픽이 많은 정적 리소스에 우선적으로 활용하고, 상대적으로 덜 중요한 리소스는 S3만 사용하여 하이브리드 전략을 채택합니다.

