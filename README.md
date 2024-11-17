## 목차
- [배포 파이프라인](#배포-파이프라인)
- [github action](#github-action)
- [주요 링크](#주요-링크)
- [주요 개념](#주요-개념)
- [CDN과 성능최적화/성능개선 보고서](#cdn과-성능최적화)

## 배포 파이프라인

### 개요
![hanghae-plus (2)](https://github.com/user-attachments/assets/18695f1f-ccff-428d-9326-be4f9ce5ac4a)
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
  - 캐시 무효화는 어던 시점에 동작하는가?
    - 명시적으로 캐시 무효화를 요청한 경우(명령어 ```aws cloudfront create-invalidation```)
    - 캐시 만료 시간에 따라 자동으로 동작(기본적으로 24시간으로 설정)
    - 파일변경으로 인해 ETag 또는 Last-Modified 헤더가 달라질때
    - CloudFront 배포 업데이트 시점
- Repository secret과 환경변수
  - 환경변수 : 실행환경에서 어플리케이션에서 전달되는 값으로 설정값, 경로 정보, 변수 등을 저장하는데 사용된다.
  - Repository secret : 민감한 데이터를 보호하기 위해 github actions와 같은 CI/CD도구에서 사용하는 보안 환경 변수이다. 주로 API 키, 클라우드 자격증명 등 민감한 정보를 저장한다.

## CDN과 성능최적화
### CDN 도입 전 vs 도입 후 
<img width="1108" alt="image" src="https://github.com/user-attachments/assets/484e731e-0e9d-409b-b3ec-05e2f551965a">
크롬 개발자도구 > 네트워크 탭에서 확인. 왼쪽이 S3로 배포했을 때, 오른쪽이 CloudFront로 배포했을때(CDN 적용).

- 측정 지표
  - 로드 시간
  - 첫번째 바이트 응답 시간
- 성능 비교 분석
  - 페이지 로드 시간(Total Load Time, **67% 개선**)
    - S3 : 3초
    - CloudFront : 1초
  - TTFB(Time to First Byte, **73% 개선**)
    - S3 : 1.5초
    - CloudFront : 0.4초
- 결론
  - ColudFront를 사용하여 CDN을 활용하는 것이 웹사이트 로드 시간을 단축시켜 사용자의 경험을 높일 수 있어 ColudFront을 사용하는 것이 좋음.