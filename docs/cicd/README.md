# CI/CD 가이드

프로젝트의 CI/CD 파이프라인 구성과 배포 전략을 다룹니다.

## 목차

- [GitHub Actions](./github-actions.md) - GitHub Actions 워크플로우 설정
- [Azure Pipelines](./azure-pipelines.md) - Azure DevOps 파이프라인 설정  
- [Docker 배포](./docker-deployment.md) - Docker를 활용한 배포 전략
- [품질 관리](./quality-pipeline.md) - CI/CD에서의 코드 품질 검증
- [자동화 도구](./automation-tools.md) - 의존성 관리 및 릴리스 자동화

## 개요

이 디렉토리는 CI/CD 파이프라인 구성에 필요한 **실용적인 설정과 전략**을 다룹니다.

### 지원 플랫폼

1. **GitHub Actions**: GitHub 호스팅 프로젝트 (기본 추천)
2. **Azure Pipelines**: Azure DevOps 환경
3. **Docker**: 컨테이너 기반 배포

### 핵심 원칙

1. **간결성**: 복잡한 설정보다는 실용적이고 유지보수 가능한 구성
2. **유연성**: GitHub Actions와 Azure Pipelines 모두 지원
3. **표준화**: 팀 전체가 사용할 수 있는 일관된 패턴

## 빠른 시작

### GitHub Actions 기본 설정

```bash
# .github/workflows/ 디렉토리 생성
mkdir -p .github/workflows

# 기본 워크플로우 파일 생성
cp cicd/templates/github-basic.yml .github/workflows/ci.yml
```

### Azure Pipelines 기본 설정

```bash
# Azure Pipelines 설정 파일 생성
cp cicd/templates/azure-basic.yml azure-pipelines.yml
```

## 플랫폼별 특징

| 플랫폼            | 장점                    | 단점                | 권장 사용 사례            |
| ----------------- | ----------------------- | ------------------- | ------------------------- |
| **GitHub Actions** | 간편 설정, GitHub 통합 | GitHub 의존성       | 오픈소스, 일반 프로젝트   |
| **Azure Pipelines** | 강력한 기능, 멀티 플랫폼 | 설정 복잡성         | 기업 환경, 복잡한 워크플로우 |
| **Docker**        | 환경 일관성             | 초기 설정 복잡      | 마이크로서비스, 컨테이너 환경 |

## 권장 워크플로우

1. **코드 품질 검증**: ESLint, Prettier, TypeScript 타입 체크
2. **테스트 실행**: 단위 테스트, 통합 테스트
3. **빌드**: 프로덕션 빌드 및 아티팩트 생성
4. **배포**: 환경별 자동 배포

## 참고 자료

- [GitHub Actions 공식 문서](https://docs.github.com/en/actions)
- [Azure Pipelines 공식 문서](https://docs.microsoft.com/en-us/azure/devops/pipelines/)
- [Docker 공식 문서](https://docs.docker.com/)