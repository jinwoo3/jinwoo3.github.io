---
title: Kubernetes 환경 CD(Continuous Deployment) 개선하기
author: derek
tags:
  - Development
excerpt: About Kubernetes CD
date: 2020-12-18
featuredImage: ./images/1.png
---

안녕하세요. 숨고 DevOps 엔지니어 Derek 입니다.

최근 진행한 Kubernetes 환경에 대한 CD(Continuous Deployment)개선에 대하여 개선 이유, 작업 내용, 결과를 포스팅 해보려 합니다.

동일한 고민을 갖고있을 DevOps 엔지니어 또는 개발자에게 조금이나마 도움이 되길 바랍니다.

# 1. 배경 설명

숨고 서비스를 지탱하는 Application 중 다수의 것들이 Kubernetes 환경에서 운영되고 있습니다. MSA Architecture 로 설계 되어 있으며, 때문에 잘게 나누어진 Application Repository 가 존재합니다. Kubernetes 에 배포를 위한 Manifest 파일은 해당 Application Repository 에 함께 존재했습니다.
숨고는 CI/CD 를 위해 Bitbucket Pipeline 을 사용하고 있습니다. 때문에 도커 이미지 빌드, Kubernetes 배포 또한 Bitbucket Pipeline 을 통하여 진행됩니다. 또한 무 중단 배포를 위한 Blue/Green 배포 스크립트가 존재하고 Bitbucket Pipeline 에서 해당 스크립트를 실행하여 배포하게 됩니다.

# 2. 개선 이유

### 2–1. Application Code 와 Manifest File 의 Repository 분리

앞서 배경 설명에서 언급 하였듯이 Application Code 와 Manifest 파일이 동일한 Repository 에서 관리 되었습니다. 이는 다음과 같은 단점을 갖고 있었고 이를 분리하여 문제를 해결하고자 했습니다.

- Manifest File 만 변경시에도 안전한 배포(무중단)를 위해서는 컨테이너 빌드 절차를 거쳐야함. (Kubernetes에서 변경이 있을경우 Rollout 되는 항목에 한하여)
- Manifest File 이 여러 Application Repository 에 분산되어 관리가 어려움.
- Application Code 변경 로그와 Manifest 변경 로그가 뒤섞여 가시성 떨어짐.
- Repository 수정 권한이 있는 모든 인원이 Manifest 파일을 수정하고 이를 Production 까지 적용할 수 있어 안정성이 떨어짐.

### 2-2. GitOps 적용

GitOps 란 Kubernetes 클러스터를 관리하고 Application 을 배포하는 방법론이며, 이것은 선언적 인프라와 Application을 위해 Git 을 단일 소스(single source of truth) 로 사용하는것을 말합니다.

즉, 설정파일 관리 Repository (Git) 을 통해서만(단일한 방법) Kuberentes 클러스터가 변경되고 Application이 배포되는 것입니다.

GitOps 적용으로 인한 장점은 다음과 같습니다.

- Git Repository의 현재 상태만 확인하면 Kubernetes 클러스터의 상태를 쉽게 확인가능.
- 모든 설정 정보 변경이 Git의 버전으로 관리됨.
- kubectl, helm 등의 명령어를 사용자가 직접 실행하는 것이 아니기 때문에 휴먼 에러를 방지하며 안정적으로 상태를 변경하고 배포할수 있음.
- Repository 권한 설정으로 Kubernetes 클러스터 접근 권한 분리 가능.

이와 같은 장점을 위해 GitOps를 적용하고자 했습니다.

### 2-3. 배포 모니터링 개선

현 배포 시스템의 경우(Bitbucket Pipeline) Kubernetes에 명령을 실행한후 적용이 완료 되었는지 또는 실패 하였는지 등의 모니터링이 되지 않습니다.

ArgoCD 를 사용하여 배포 모니터링을 개선하고자 하였습니다.

# 3. 작업 내용

### 3–1. 개선 방향 설정

앞서 얘기한 현시점의 배포 문제점과 새로 제안한 개선점들을 통하여 실제 작업 내용을 Flow Chart로 만들어봤습니다.

아래 As-Is, To-Be Flow 그림에서 확인할 수 있듯이 Repository 를 분리하고 Manifest Repository 에 변경이 발생하였을때 ArgoCD 에서 이를 감지하여 Sync (배포 or 변경) 하고자 하였습니다.

1. As-Is flow

![](./images/1.jpeg)

2. To-Be flow

![](./images/2.jpeg)

### 3-2. Manifest Repository 생성

Application Respository 에서 Manifest 를 분리하기 위해 통합된 Manifest Repository 를 생성하였습니다. 해당 Repository 는 Helm Chart 를 사용하여 구조화 하였습니다.

```
├── api-gateway #Application 별 Directory
│ ├── Chart.yaml
│ ├── charts
│ ├── dev.values.yaml #환경별 Values 파일
│ ├── prod.values.yaml
│ ├── qa.values.yaml
│ ├── staging.values.yaml
│ ├── templates #Application Templates
│ │ ├── deployment.yaml
│ │ ├── hpa.yaml
│ │ ├── ingress.yaml
│ │ ├── service-standby.yaml
│ │ └── service.yaml
│ └── test.values.yaml
....
```

해당 Repository 는 배포 환경별 Branch 가 존재합니다. 예를 들어 Staging 배포를 위한 staging branch, QA 환경 배포를 위한 QA branch, Production 배포를 위한 Master branch 와 같은 형태입니다.

### 3-2. ArgoCD, Argo-rollouts, ArgoCD-notifications 사용

앞서 언급 하였듯이 배포 개선을 하기위하여 GitOps 의 구현체로 ArgoCD 를 사용하기로 결정했습니다. 여러 다양한 도구중 ArgoCD 를 사용한 이유는 설치 및 사용이 쉽고 간편 했기 때문입니다.

또한 GitOps 구현체로서 반드시 필요한 기능만 갖고 있는 가벼운 느낌이 있었습니다. 또한 참고할수 있는 Reference 역시 많았습니다.

ArgoCD 뿐만 아니라 Blue/Green, Canary 등 배포 전략을 사용하기위해 Argo-rollouts 를 사용하였고, ArgoCD 의 Sync 상태와 등록된 Application의 상태를 Slack 을 통해 알림 받을수 있도록 ArgoCD-notification 도 사용하였습니다.

- ArgoCD: https://argoproj.github.io/argo-cd/
- Argo-rollouts: https://argoproj.github.io/argo-rollouts/
- ArgoCD-notifications: https://argoproj.github.io/argo-cd/operator-manual/notifications/

각각에 대한 문서는 위와 같으며, 설치 방법이 쉽게 설명되어 있습니다.

# 3. ArgoCD, Manifest Repository 연동

ArgoCD 에 등록한 Application은 배포 목적지에 따라 그에 맞는 Branch, Directory 와 sync 하고 환경별 Values 파일을 참조합니다.

예를 들어 admin-api-gateway-qa Application 은 QA Branch 의 admin-api-gateway 디렉토리에 변경사항이 생겼을 경우 Sync 하며, qa.values.yaml 파일을 참조하여 Sync 합니다.

![](./images/3.png)

![](./images/4.png)

# 4. Application Repository 와 Manifest Repository 연동

앞서 개선 방향 그림에서 설명하였듯이 Application Repository 의 Pipeline 은 빌드를 담당하고 빌드이후 Manifest Repository에 변경사항(이미지 tag) 을 적용하여야 합니다. 따라서 다음과 같은 명령어를 사용하여 Pipeline 에서 Manifest Repository 에 Commit 하도록 하였습니다.

```zsh
- git clone https://$GIT_USER_NAME:$GIT_USER_PASSWORD@bitbucket.org/soomgo/kubernetes-manifest.git && cd kubernetes-manifest
- git config --global user.email $GIT_USER_EMAIL; git config --global user.name $GIT_USER_NAME ; git config --global push.default matching
- git fetch && git checkout qa
- sed -i 's/tag:.\*/tag:'' '\$BITBUCKET_COMMIT_SHORT'/g'
- \$CONTAINER_APP_NAME/qa.values.yaml
- if [ -z "$(git status --porcelain)" ];
  then
  echo "nothing to commit, working directory clean";
  else
  git commit -am "https://bitbucket.org/"$BITBUCKET_COMMIT_LINK && git push;
  fi
```

# 결과

### 1. Application Repository 와 Manifest File 의 Repository 완전히 분리

각각의 Repository 가 완전히 분리됨에 따라 기존의 문제점들이 사라졌습니다. 여러 Application 들의 Manifest File 들이 한 Repository 에서 관리 되기 때문에 설정파일 관리 측면에서도 이전보다 많이 수월해 졌습니다. 또한 Master Branch 에 대한 권한 설정으로 Production 환경 관리 측면에서 안정성을 높일수 있게 되었습니다.

### 2. GitOps 적용 완료

Kubernetes 환경은 하나의 출처(source)인 Manifest Repository 에 의해서만 관리되게 되었습니다. 기존처럼 변경을 위해 CLI 를 직접 실행하는 일을막고 Git Commit 이 이를 대체할수 있게 되었습니다.

### 3. 배포 모니터링 개선 완료

ArgoCD 와 ArgoCD-notifications 를 사용 함으로서 배포에 대한 모니터링이 개선 되었습니다.
아래 그림처럼 ArgoCD 를 통해 상태에 대한 모니터링이 가능하며, 상태에 대한 알림을 Slack 을 통해 받을수 있습니다.

![](./images/5.png)

![](./images/6.jpeg)

![](./images/7.png)

# 맺음말

CD (Continuous Deployment) 를 개선한 이유와 개선 방법, 결과에 대한 내용을 작성해 봤습니다. 모든 개선이 끝이 없는것처럼 위 내용도 끝이 아닌 새로운 시작인것 같습니다. 좀 더 운영해보며 문제점들을 찾아 지속적인 개선을 할 계획이고 CD 뿐 아니라 CI 측면에서도 지속적인 개선을 해나갈 생각입니다.

이후 개선작업 관련해서 공유할만 주제가 있다면 숨고 기술블로그를 통해 포스팅할 예정입니다.

부족한 글 읽어주셔서 감사합니다.
