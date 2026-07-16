# Azure Pipelines 설정

Azure DevOps를 활용한 CI/CD 파이프라인 구성 가이드입니다.

## 기본 파이프라인

### azure-pipelines.yml

```yaml
# Azure Pipelines 기본 설정
trigger:
  branches:
    include:
      - main
      - develop

pr:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  nodeVersion: '18.x'
  pnpmVersion: '8.15.0'

stages:
  - stage: Quality
    displayName: 'Code Quality'
    jobs:
      - job: QualityCheck
        displayName: 'Quality Check'
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: $(nodeVersion)
            displayName: 'Setup Node.js'

          - script: |
              npm install -g pnpm@$(pnpmVersion)
              pnpm install --frozen-lockfile
            displayName: 'Install dependencies'

          - script: pnpm lint
            displayName: 'Lint check'

          - script: pnpm type-check
            displayName: 'Type check'

          - script: pnpm test --coverage
            displayName: 'Run tests'

          - task: PublishTestResults@2
            condition: succeededOrFailed()
            inputs:
              testResultsFiles: 'coverage/junit.xml'
              testRunTitle: 'Unit Tests'

          - task: PublishCodeCoverageResults@1
            inputs:
              codeCoverageTool: 'Cobertura'
              summaryFileLocation: 'coverage/cobertura-coverage.xml'

  - stage: Build
    displayName: 'Build'
    dependsOn: Quality
    condition: succeeded()
    jobs:
      - job: BuildApp
        displayName: 'Build Application'
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: $(nodeVersion)

          - script: |
              npm install -g pnpm@$(pnpmVersion)
              pnpm install --frozen-lockfile
            displayName: 'Install dependencies'

          - script: pnpm build
            displayName: 'Build application'

          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: 'dist'
              artifact: 'build-files'

  - stage: Deploy
    displayName: 'Deploy'
    dependsOn: Build
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployProduction
        displayName: 'Deploy to Production'
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: DownloadPipelineArtifact@2
                  inputs:
                    artifact: 'build-files'
                    path: '$(Pipeline.Workspace)/dist'

                - script: |
                    # 배포 스크립트 실행
                    ./scripts/deploy.sh
                  displayName: 'Deploy application'
```

## 모노레포 파이프라인

### 변경 감지 및 선택적 빌드

```yaml
# azure-pipelines-monorepo.yml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - packages/*
      - apps/*

variables:
  nodeVersion: '18.x'
  pnpmVersion: '8.15.0'

stages:
  - stage: DetectChanges
    displayName: 'Detect Changes'
    jobs:
      - job: DetectChanges
        displayName: 'Detect Changed Packages'
        steps:
          - script: |
              # 변경된 패키지 감지 스크립트
              git diff --name-only HEAD~1 HEAD | grep -E '^(packages|apps)/' > changed-paths.txt || true
              echo "##vso[task.setvariable variable=hasChanges;isOutput=true]$([ -s changed-paths.txt ] && echo true || echo false)"
            name: detectChanges
            displayName: 'Detect changes'

  - stage: TestPackages
    displayName: 'Test Packages'
    dependsOn: DetectChanges
    condition: eq(dependencies.DetectChanges.outputs['detectChanges.hasChanges'], 'true')
    jobs:
      - job: TestPackages
        displayName: 'Test Changed Packages'
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: $(nodeVersion)

          - script: |
              npm install -g pnpm@$(pnpmVersion)
              pnpm install --frozen-lockfile
            displayName: 'Install dependencies'

          - script: pnpm test --filter="./packages/*"
            displayName: 'Test packages'

  - stage: BuildApps
    displayName: 'Build Apps'
    dependsOn: TestPackages
    condition: succeeded()
    jobs:
      - job: BuildApps
        displayName: 'Build Applications'
        strategy:
          matrix:
            web-app:
              appName: 'web-app'
            admin-dashboard:
              appName: 'admin-dashboard'
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: $(nodeVersion)

          - script: |
              npm install -g pnpm@$(pnpmVersion)
              pnpm install --frozen-lockfile
            displayName: 'Install dependencies'

          - script: pnpm build --filter=@project/$(appName)
            displayName: 'Build $(appName)'

          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: 'apps/$(appName)/dist'
              artifact: '$(appName)-build'
```

## Docker 통합

### Azure Container Registry

```yaml
# Docker 빌드 및 푸시
- stage: Docker
  displayName: 'Docker Build'
  jobs:
    - job: DockerBuild
      displayName: 'Build Docker Image'
      steps:
        - task: Docker@2
          displayName: 'Build and push image'
          inputs:
            containerRegistry: 'myacr'
            repository: 'myapp'
            command: 'buildAndPush'
            Dockerfile: '**/Dockerfile'
            tags: |
              $(Build.BuildNumber)
              latest
```

## 환경별 배포

### 환경 설정

```yaml
# 환경별 변수 그룹 사용
variables:
  - group: 'production-variables'  # Azure DevOps 변수 그룹
  - name: 'environment'
    value: 'production'

# 조건부 배포
- stage: DeployProduction
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
    - deployment: ProductionDeploy
      environment: 'production'
      strategy:
        runOnce:
          deploy:
            steps:
              - script: |
                  echo "Deploying to $(environment)"
                  ./scripts/deploy.sh $(environment)
```

## 보안 및 승인

### 승인 게이트

```yaml
# 프로덕션 배포시 수동 승인 필요
- stage: DeployProduction
  jobs:
    - deployment: ProductionDeploy
      environment: 'production'  # 환경에 승인 규칙 설정
      strategy:
        runOnce:
          deploy:
            steps:
              - script: ./deploy-production.sh
```

### 시크릿 관리

```yaml
# Azure Key Vault 연동
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'my-subscription'
    KeyVaultName: 'my-keyvault'
    SecretsFilter: 'database-password,api-key'

- script: |
    echo "Database password: $(database-password)"
    echo "API key: $(api-key)"
```

## 성능 최적화

### 캐시 설정

```yaml
# 의존성 캐시
- task: Cache@2
  inputs:
    key: 'pnpm | "$(Agent.OS)" | pnpm-lock.yaml'
    path: '~/.pnpm-store'
    restoreKeys: |
      pnpm | "$(Agent.OS)"
  displayName: 'Cache pnpm store'

# 빌드 캐시  
- task: Cache@2
  inputs:
    key: 'build | "$(Agent.OS)" | package.json | **/*.ts | **/*.tsx'
    path: |
      .turbo
      **/dist
  displayName: 'Cache build outputs'
```

### 병렬 처리

```yaml
# 여러 작업 병렬 실행
jobs:
  - job: Lint
    steps:
      - script: pnpm lint

  - job: TypeCheck
    steps:
      - script: pnpm type-check

  - job: Test
    steps:
      - script: pnpm test

  - job: Build
    dependsOn: [Lint, TypeCheck, Test]
    steps:
      - script: pnpm build
```

## 실용적인 팁

### 조건부 실행

```yaml
# 파일 변경 기반 조건부 실행
- script: |
    if git diff --name-only HEAD~1 HEAD | grep -q "packages/"; then
      echo "##vso[task.setvariable variable=packagesChanged]true"
      pnpm test --filter="./packages/*"
    fi
  displayName: 'Test packages if changed'
```

### 알림 설정

```yaml
# Teams 알림
- task: TeamsFx@0
  condition: failed()
  inputs:
    webhookUrl: '$(teamsWebhookUrl)'
    message: 'Pipeline failed for $(Build.SourceBranch)'
```

## 템플릿

### 기본 파이프라인

```yaml
# templates/azure-basic.yml
trigger: [main, develop]
pr: [main]

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '18.x'
      
  - script: |
      npm install -g pnpm
      pnpm install --frozen-lockfile
    displayName: 'Install dependencies'
    
  - script: pnpm lint && pnpm test && pnpm build
    displayName: 'CI Pipeline'
```

## GitHub Actions vs Azure Pipelines

| 기능              | GitHub Actions | Azure Pipelines |
| ----------------- | -------------- | --------------- |
| **설정 파일**     | .yml           | .yml            |
| **환경 관리**     | Environments   | Environments    |
| **시크릿 관리**   | Secrets        | Variable Groups |
| **승인 프로세스** | 환경 보호 규칙 | 승인 게이트     |
| **캐시**          | actions/cache  | Cache@2         |
| **아티팩트**      | upload/download| PublishArtifact |