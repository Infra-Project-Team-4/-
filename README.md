# GitLab CI/CD와 Ansible을 활용한 자동화 창고관리시스템 구축

# 프로젝트 목적

본 프로젝트는 GitLab CI/CD와 Ansible을 활용한 자동화 창고관리시스템(WMS) 구축 프로젝트이다.

GitLab Pipeline을 통해 소스 코드 관리부터 빌드, 테스트, 배포까지의 CI/CD 프로세스를 자동화하고, Ansible을 이용해 서버 환경 구성 및 애플리케이션 배포를 코드 기반으로 관리한다.

이를 통해 반복적인 수동 작업을 최소화하고, 안정적이고 일관된 운영 환경을 구축하여 창고관리시스템의 배포 효율성과 유지보수성을 향상시키는 것을 목표로 한다.

# 구현 범위

<img width="1000" height="600" alt="image" src="https://github.com/user-attachments/assets/f53cd3ee-d943-4cd0-93fa-f7cc3c5ae210" />



# 담당 역할
[앤서블을 이용하여 서버 자동 구축 작업 기록](https://app.notion.com/p/38ba48359dbb80579320e8e0cb68c6cd) 
<br>

```
Role 구성 → Inventory 작성 → 변수 설정 → Playbook 작성 → site.yml 작성(전체 실행)→ ansible-playbook 실행
→ 서버 자동 구축 완료
```

[깃랩 CI/CD 파이프라인 구축 작업 기록 ](https://app.notion.com/p/CI-CD-38fa48359dbb80f387dbf4413371dab0)
<br>

```
Git Repository 구성 → .gitlab-ci.yml 작성 → Pipeline Stage 구성 → Build → Test →Docker Image Build
→ Harbor Push → Kubernetes Deploy → Pod Rolling Update
```
                      

# 기술 스택

<img width="1536" height="1024" alt="ChatGPT Image 2026년 6월 30일 오전 11_37_50" src="https://github.com/user-attachments/assets/de9642fa-0ad4-47f3-bbfc-8a6232e6169e" />


# 네트워크 구성 (Logical Architecture)

<img width="200" height="200" alt="pfsense" src="https://github.com/user-attachments/assets/c2241ae6-a962-4688-82a6-cc3fa011e0f7" />   ## Pfsense 적용 



# Ansible 자동화 구성

```
ansible-infra/
│
├── ansible.cfg
│   └── Ansible 환경 설정
│
├── playbooks/
│   ├── site.yml
│   │   └── 전체 인프라 구축 실행
│   │
│   ├── init-server.yml
│   │   └── 서버 초기 설정
│   │
│   ├── kubernetes-install.yml
│   │   └── Kubernetes 설치
│   │
│   └── deploy.yml
│       └── 서비스 배포
│
└── roles/
    │
    ├── database/
    │   └── PostgreSQL / Redis / Kafka 구축
    │
    ├── docker/
    │   └── Docker 환경 구성
    │
    ├── harbor/
    │   └── Private Registry 구성
    │
    └── kubernetes/
        ├── Master Node 구성
        ├── Worker Node Join
        └── Cluster 환경 구축
```

# CI/CD Pipeline
(GitLab Pipeline 이미지)


# 서비스 구성
(DB / Redis / Kafka 등)

# 트러블슈팅 / 장애 해결

# 결과 및 성과

# 느낀점
