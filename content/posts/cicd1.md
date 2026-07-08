---
title: "CI/CD 자동화 - 1"
date: 2026-07-07T02:18:00+09:00
draft: false
categories: ["DevOps", "CI/CD"]
---


**CI/CD 자동화 — 1. 개요와 기본 흐름**

소스 코드를 깃허브 레포지토리에 **Push**하면, 미리 정의해둔 GitHub Actions 워크플로가 자동으로 실행되어 **지속적 통합(Continuous Integration)**과 **지속적 배포(Continuous Deployment)**를 수행한다. 이번 글은 시리즈의 첫 번째 글로서 CI/CD의 개념을 간단히 정리하고, GitHub Actions가 어떻게 동작하는지, 그리고 가장 기본적인 워크플로 예시를 통해 전체 흐름을 이해하는 것을 목표로 한다.

---

### CI와 CD, 그리고 자동화의 가치

- **CI(Continuous Integration)**: 개발자가 변경한 코드를 중앙 저장소에 자주 병합하고, 자동화된 빌드와 테스트를 통해 통합 문제를 조기에 발견하는 프로세스다. 지속적인 통합으로 일반적인 통합 대비 통합 시 발생하는 비용발생을 줄일 수 있다.  
- **CD(Continuous Delivery / Continuous Deployment)**: CI를 통과한 결과물을 자동으로 스테이징 또는 프로덕션 환경에 배포하는 단계다.  
- **자동화의 가치**: 수동 작업을 줄여 배포 속도를 높이고, 사람 실수로 인한 오류를 줄이며, 빠른 피드백 루프를 만들어 개발 생산성을 향상시킨다.

---

### GitHub Actions가 하는 일 — 핵심 흐름

1. **Push 이벤트 발생**  
   개발자가 로컬에서 커밋 후 `git push`를 하면 GitHub가 해당 이벤트를 감지한다.

2. **워크플로 트리거**  
   저장소의 `.github/workflows/` 디렉터리에 있는 YAML 파일 중, 이벤트 조건에 맞는 워크플로가 실행된다.

3. **런너에서 작업 실행**  
   워크플로는 여러 **job**과 **step**으로 구성된다. 각 job은 GitHub-hosted runner 또는 self-hosted runner에서 실행되며, 빌드, 테스트, 린트, 패키징, 배포 등의 작업을 수행한다. 추후에 자세히 다룰예정.

4. **아티팩트 생성 및 배포**  
   빌드 결과물(정적 사이트, 바이너리, 컨테이너 이미지 등)을 생성하고, 필요하면 아티팩트로 업로드하거나 외부 서비스(예: GitHub Pages, S3, Docker Registry)에 배포한다. Rsync, scp 등으로 서버에 직접 전달할 수도 있고 GHCR, 도커허브에 배포하고 서버장비에 SSH를 통한 `pull`명령을 내리기도 가능하다.

5. **알림 및 피드백**  
   빌드 실패나 배포 성공 여부는 PR 코멘트, 이메일, 슬랙 등으로 알림을 보낼 수 있다.

---

### 간단한 예시 워크플로 (Hugo 정적 사이트 빌드 + deploy 폴더 생성)

아래 예시는 저장소 루트에서 Hugo 소스가 `./build`에 있고, 빌드 결과를 `./deploy`로 생성하는 기본적인 워크플로다. (실제 배포 스텝은 추후 자세히 기술)

```yaml
name: Build and Prepare Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build site
        run: hugo --minify --source ./build --destination ./deploy

      - name: Upload deploy artifact
        uses: actions/upload-artifact@v4
        with:
          name: site-deploy
          path: ./deploy
```

이 워크플로는 Push → Build → deploy 폴더(결과물) 생성 → 아티팩트 업로드의 기본 흐름을 보여준다. 실제 배포(예: gh-pages, S3 업로드, FTP 전송 등)는 Upload deploy artifact 이후에 추가 스텝으로 연결하면 된다.

### 장점과 고려사항

#### 장점

- **자동화로 일관성 확보**: 모든 빌드는 동일한 환경에서 수행되어 "내 로컬에서는 되는데" 문제를 줄인다.
- **빠른 피드백**: 테스트 실패나 빌드 오류를 즉시 확인할 수 있어 문제 해결이 빨라진다.
- **배포 속도 향상**: 수동 배포 단계를 제거하면 릴리스 주기가 빨라진다.

#### 주의할 점

- **비밀(Secrets) 관리**: 배포 키, 토큰 등 민감 정보는 GitHub Secrets에 안전하게 저장해야 한다. 관련내용은 추후 다룰 예정.
- **작업 시간/비용**: 호스티드 러너 사용 시 빌드 시간이 길어지면 비용과 대기 시간이 늘어난다.
- **환경 재현성**: 로컬과 CI 환경의 차이를 줄이기 위해 고정된 툴 버전을 사용하거나(비추) 도커 컨테이너(강추)를 사용하라.

### 마무리

이번 글은 CI/CD 자동화의 서론으로, GitHub Actions가 어떻게 소스 Push 한 번으로 빌드와 배포 파이프라인을 트리거하는지 기본 흐름을 정리했다. 다음 글부터는 실전에서 바로 적용 가능한 워크플로 패턴과 팁이나 CI/CD자동화의 세부항목을 하나씩 풀어갈테니, 많은 관심 바랍니다.