# Docker 가이드

서버 기반 애플리케이션을 위한 Docker 설정 및 배포 가이드입니다.

## 🤔 Docker가 언제 필요한가요?

### ✅ Docker가 필요한 경우

- **Next.js**: SSR/SSG가 필요한 경우
- **Nest.js**: API 서버 및 백엔드 서비스
- **Express.js**: Node.js 백엔드 서버
- **Database**: PostgreSQL, MongoDB 등
- **Redis**: 캐시 서버
- **Python/Django**: 웹 서버
- **Go/Rust**: 백엔드 API 서버

### ❌ Docker가 불필요한 경우

- **Pure React**: 빌드 후 정적 파일만 배포 (Nginx, Vercel, Netlify 등)
- **Vue.js SPA**: 정적 호스팅으로 충분
- **Angular SPA**: CDN 배포로 충분

## 📁 구조

```
docker/
├── README.md              # 이 파일
├── nextjs.md             # Next.js 앱용 Docker 설정
├── nestjs.md             # Nest.js API용 Docker 설정
└── database.md           # 데이터베이스 컨테이너 설정
```

## 🚀 빠른 시작

각 애플리케이션 타입별로 적절한 가이드를 참조하세요:

- **Next.js 앱**: `nextjs.md` 참조
- **Nest.js API**: `nestjs.md` 참조
- **데이터베이스**: `database.md` 참조

## 📚 추가 리소스

- [Docker 공식 문서](https://docs.docker.com/)
- [Next.js with Docker 예제](https://github.com/vercel/next.js/tree/canary/examples/with-docker)
- [Nest.js Docker 가이드](https://docs.nestjs.com/recipes/docker)
