# CDN 도입 성능 개선 보고서

## 개요
![image](https://github.com/user-attachments/assets/1a2accf0-77ff-45a1-9dd5-ff41f92c6bd3)

GitHub Actions을 사용해 애플리케이션을 AWS S3 및 CloudFront에 배포하는 워크플로우 입니다.

## 배포 워크플로우

1. 저장소 체크아웃
2. Node.js 18.x 버전 설정
3. 프로젝트 의존성 설치
4. Next.js 프로젝트 빌드
5. AWS 자격 증명 구성
6. 빌드된 파일을 S3 버킷에 동기화
7. CloudFront 캐시 무효화

### GitHub Actions YAML 파일 예시
```yaml
name: Deploy Next.js to S3 and invalidate CloudFront

on:
  push:
    branches:
      - main  
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Install dependencies
      run: npm ci

    - name: Build
      run: npm run build

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}

    - name: Deploy to S3
      run: |
        aws s3 sync out/ s3://${{ secrets.S3_BUCKET_NAME }} --delete

    - name: Invalidate CloudFront cache
      run: |
        aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
```



## 주요 링크
- S3 버킷 웹사이트 엔드포인트: [http://hanghae-2estella.s3-website-us-east-1.amazonaws.com/](http://hanghae-2estella.s3-website-us-east-1.amazonaws.com/)
- CloudFront 배포 도메인 이름: [https://d2cvvqmbjry9rr.cloudfront.net/](https://d2cvvqmbjry9rr.cloudfront.net/)


## 주요 개념
- **GitHub Actions과 CI/CD 도구**:
  - GitHub Actions는 저장소 변경 사항을 감지해 자동으로 빌드 및 배포할 수 있는 CI/CD 도구입니다. 이를 통해 코드 변경 사항을 빠르고 안정적으로 배포할 수 있으며, 반복적인 수작업을 자동화할 수 있습니다.
  - 사용 이유: GitHub Actions는 저장소 변경 시 자동으로 빌드 및 배포 작업을 실행할 수 있어, 코드 배포 과정을 자동화하고 수작업을 줄여줍니다. 이는 개발 생산성을 높이고, 배포의 일관성과 안정성을 제공합니다.
  - 사용 방법: GitHub Actions의 워크플로우를 정의하여, push 또는 pull request 이벤트에 따라 자동으로 빌드, 테스트, 배포 작업을 실행합니다. YAML 파일로 설정을 작성하고, CI/CD 프로세스를 정의할 수 있습니다.
- **S3와 스토리지**:
  - AWS S3(Simple Storage Service)는 정적 웹사이트 호스팅을 지원하는 클라우드 스토리지 서비스로, 이미지, CSS, JavaScript 파일 등의 정적 파일을 효율적으로 저장하고 제공하는 데 사용됩니다. 높은 내구성과 확장성을 갖춘 스토리지 솔루션입니다.
  - 사용 이유: AWS S3는 정적 웹사이트 호스팅을 지원하고, 이미지, CSS, JavaScript 파일을 효율적으로 저장 및 제공할 수 있어, 대규모 스토리지와 확장성을 제공합니다. 비용 효율적이고 신뢰성이 높습니다.
  - 사용 방법: AWS S3에 정적 파일을 업로드하고, 버킷을 웹사이트 호스팅용으로 설정하여 파일을 제공할 수 있습니다. S3의 관리 콘솔 또는 AWS CLI를 사용해 파일을 관리하고, 퍼블릭 액세스를 허용합니다.
- **CloudFront와 CDN**:
  - AWS CloudFront는 글로벌 엣지 서버를 활용해 콘텐츠를 캐싱하고 사용자에게 더 빠르게 제공하는 CDN(Content Delivery Network) 서비스입니다. 정적 파일뿐만 아니라 동적 콘텐츠도 가속화할 수 있으며, 보안 기능(예: DDoS 보호)도 제공합니다.
  - 사용 이유: AWS CloudFront는 글로벌 엣지 서버를 사용해 콘텐츠를 빠르게 전달하며, 정적 및 동적 콘텐츠를 가속화할 수 있습니다. 또한, DDoS 방어와 같은 보안 기능을 제공해 신뢰성도 높습니다.
  - 사용 방법: CloudFront 배포를 생성하여 S3 버킷과 연결하고, 사용자에게 더 가까운 엣지 서버에서 콘텐츠를 제공하도록 설정합니다. CloudFront는 자동으로 콘텐츠를 캐싱하고, 빠른 로딩 속도를 제공합니다.
- **캐시 무효화(Cache Invalidation)**:
  - CloudFront에 캐싱된 오래된 파일을 제거해 최신 버전이 사용자에게 제공되도록 하는 프로세스입니다. 이를 통해 코드 배포 후 사용자들이 변경된 내용을 즉시 확인할 수 있도록 합니다.
  - 사용 이유: 캐시 무효화는 CloudFront에 캐시된 오래된 파일을 제거하고 최신 파일을 즉시 제공할 수 있게 해줍니다. 이를 통해 배포 후 사용자에게 최신 콘텐츠를 빠르게 반영할 수 있습니다.
  - 사용 방법: CloudFront 콘솔 또는 AWS CLI를 통해 캐시 무효화 요청을 보내고, 변경된 파일에 대해 캐시를 새로 고칩니다. 주기적으로 또는 특정 파일에 대해 캐시를 무효화합니다.
- **Repository secret과 환경변수**:
  - GitHub Secrets 기능을 사용하면 AWS 자격 증명과 같은 민감한 데이터를 안전하게 저장하고 활용할 수 있습니다. 이를 통해 CI/CD 프로세스에서 보안성을 높일 수 있습니다.
  - 사용 이유: GitHub Secrets를 사용하면 AWS 자격 증명 등 민감한 정보를 안전하게 저장할 수 있습니다. 이를 통해 CI/CD 과정에서 민감한 데이터를 보호하고 보안성을 강화할 수 있습니다. 
  - 사용 방법: GitHub 리포지토리의 Settings > Secrets에서 환경 변수를 추가하고, 워크플로우에서 이를 참조할 수 있도록 설정합니다. 예를 들어, AWS 자격 증명을 AWS_ACCESS_KEY_ID와 AWS_SECRET_ACCESS_KEY로 설정하여 배포 과정에서 사용할 수 있습니다.


## CDN과 성능 최적화
### S3 + CloudFront 최적화 전략
- 빠른 콘텐츠 제공: CloudFront는 S3에서 저장된 파일을 엣지 로케이션에서 캐시하여, 전 세계 사용자에게 빠르게 제공.
- 트래픽 분산: CloudFront가 엣지 로케이션에서 콘텐츠를 제공하므로, S3의 부하를 줄이고 트래픽을 분산.
- 정적 파일 캐시: S3에 저장된 파일을 CloudFront로 캐시하여 재사용, 캐시 히트율을 높이고 S3 요청을 줄임.
- 비용 최적화: CloudFront는 엣지 로케이션에서 콘텐츠를 제공하여 출력 비용을 줄이고, S3 요청 비용을 절감.
### 주요 이점
- 빠른 전송: 전 세계 사용자에게 빠르고 일관된 콘텐츠 제공.
- 고가용성 & 내구성: 안정적인 콘텐츠 제공.
- 비용 효율성: S3 요청을 줄여 비용 절감.
- 보안 강화: SSL/TLS, DDoS 보호, IAM 정책을 통한 보안 강화.

### CDN 도입 전후 성능 비교
- 스크린샷
<div align="center"> 
  <img width="45%" src="https://github.com/user-attachments/assets/df2b0194-845a-44e3-8b1a-3a8a42a17826">
  <img width="44%" src="https://github.com/user-attachments/assets/c81964f0-b716-48fb-9b45-38ff0173ff56"> 
</div>
<br />

- 응답시간 비교 
  | 파일명 | CDN 도입 전 응답 시간 | CDN 도입 후 응답 시간 |
  | --- | --- | --- |
  | main-app.js | 188ms | 23ms |
  | 970.js | 212ms | 26ms |
  | page.js | 200ms | 24ms |
  | font-1.woff2 | 580ms | 14ms |
  | font-2.woff2 | 582ms | 14ms |
  | styles.css | 193ms | 14ms |
  | webpack.js | 374ms | 15ms |
  | script-1.js | 1.00s | 21ms |
  | script-2.js | 1.79s | 23ms |
  | 이미지 (next.svg) | 379ms | 16ms |
  | 이미지 (vercel.svg) | 398ms | 15ms |
  | 이미지 (file.svg) | 388ms | 15ms |
  | 이미지 (window.svg) | 447ms | 16ms |
  | 이미지 (globe.svg) | 408ms | 16ms |

### 성능 개선 분석
1. **응답 속도 개선**
    - 모든 리소스에서 응답 시간이 **최소 10배 이상 단축**됨.
    - 예를 들어, `script-1.js`는 1.00s → 21ms, `script-2.js`는 1.79s → 23ms로 크게 감소.
    - 가장 큰 차이를 보이는 항목: **웹폰트 및 이미지 파일** (500ms 이상 → 14~16ms).
2. **페이지 로드 속도 개선**
    - 주요 리소스(`main-app.js`, `styles.css`, `webpack.js`)의 응답 시간이 20ms 내외로 감소해 **최적의 사용자 경험 제공** 가능.
    - 이미지 및 폰트 파일도 빠르게 로드돼 **렌더링 속도 증가**.
3. **네트워크 부하 감소**
    - 정적 파일을 CloudFront 엣지 로케이션에서 제공해 **서버 부담 최소화**.
    - AWS S3와 CloudFront 결합을 통해 **서버 비용 절감 및 확장성 확보**.

### 결론
- CDN 도입 후 **전체적인 응답 속도가 90% 이상 개선**됐고, 대용량 리소스(`script.js` 및 이미지)에서 가장 큰 효과를 확인.
- 사용자 체감 속도(First Contentful Paint, TTFB 등)가 크게 향상돼 SEO 및 UX에도 긍정적인 영향을 줄 것으로 예상.
- 향후 추가적인 최적화(압축, HTTP/3 활용, 캐시 정책 조정 등)를 고려하면 더욱 개선 가능.

---

## 참고 자료
- [AWS CloudFront 공식 문서](https://docs.aws.amazon.com/cloudfront/)
- [AWS S3 정적 웹사이트 호스팅](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html)
- [GitHub Actions 공식 문서](https://docs.github.com/en/actions)






