# GitLab CI/CD와 Ansible을 활용한 자동화 창고관리시스템 구축

# 프로젝트 목적

본 프로젝트는 GitLab CI/CD와 Ansible을 활용한 자동화 창고관리시스템(WMS) 구축 프로젝트이다.

GitLab Pipeline을 통해 소스 코드 관리부터 빌드, 테스트, 배포까지의 CI/CD 프로세스를 자동화하고, Ansible을 이용해 서버 환경 구성 및 애플리케이션 배포를 코드 기반으로 관리한다.

이를 통해 반복적인 수동 작업을 최소화하고, 안정적이고 일관된 운영 환경을 구축하여 창고관리시스템의 배포 효율성과 유지보수성을 향상시키는 것을 목표로 한다.

# 구현 범위

- 여러 대의 서버를 하나로 묶어 관리하는 환경(쿠버네티스) 위에서, 파이썬(Flask) 프로그램과 대용량 데이터 처리 장치(Kafka)를 연결해 만든 안전하고 끊김 없는 창고 관리 시스템(WMS)

<img width="1000" height="600" alt="image" src="https://github.com/user-attachments/assets/f53cd3ee-d943-4cd0-93fa-f7cc3c5ae210" />
<br>

**핵심 트래픽 및 데이터 흐름**
- 사용자 진입: 직원이 PC나 모바일 웹 화면으로 접속하면, NGINX라는 프로그램이 가장 먼저 요청을 받아 안전하게 내부 서버들로 나누어 보내줍니다.

- 기능별 서버 분리: 입고, 출고, 재고(위치/이동), QR 스캔 기능이 각각 별도의 독립된 방(컨테이너)에서 실행됩니다. 하나의 기능에 문제가 생기거나 사용자가 몰려도 다른 기능은 영향을 받지 않습니다.

- 데이터 저장 방식: 중요한 마스터 정보는 메인 저장소(PostgreSQL)에 안전하게 저장하고, 자주 쓰는 정보는 초고속 임시 저장소(Redis)에 두어 화면이 빠르게 뜨도록 합니다. 현장에서 QR 코드를 대량으로 찍을 때 발생하는 과부하는 중간 완충 장치(Kafka)가 받아주어 서버가 다운되는 것을 막습니다.
<br>

**프로그램 개발 및 자동 설치 과정**

- 자동 배포: 개발자가 수정된 코드를 깃랩(GitLab)에 올리면, 시스템이 알아서 이를 포장하여 안전한 보관소(Harbor)에 저장한 뒤, 실제 운영 중인 서버에 사용자가 눈치채지 못하도록 멈춤 없이 자동으로 교체 및 설치합니다.

# 담당 역할
앤서블을 이용하여 서버 자동 구축
<br>

```
Role 구성 → Inventory 작성 → 변수 설정 → Playbook 작성 → site.yml 작성(전체 실행)→ ansible-playbook 실행
→ 서버 자동 구축 완료
```

깃랩 CI/CD 파이프라인 구축
<br>

```
Git Repository 구성 → .gitlab-ci.yml 작성 → Pipeline Stage 구성 → Build → Test →Docker Image Build
→ Harbor Push → Kubernetes Deploy → Pod Rolling Update
```
                      

# 기술 스택

<img width="1536" height="1024" alt="ChatGPT Image 2026년 6월 30일 오전 11_37_50" src="https://github.com/user-attachments/assets/de9642fa-0ad4-47f3-bbfc-8a6232e6169e" />


# 네트워크 구성 (Logical Architecture)

<img width="200" height="200" alt="pfsense" src="https://github.com/user-attachments/assets/c2241ae6-a962-4688-82a6-cc3fa011e0f7" />
<br>

Pfsense으로 적용한 프로토콜
- 방화벽

**서비스 네트워크 토폴로지 구성도**



# Ansible 자동화 구성



Ansible은 Playbook을 실행하여 여러 Role을 순차적으로 호출하고, 각 Role이 담당하는 서버 구성을 자동으로 수행하는 구조로 구성되어 있습니다.

**Ansible 동작 과정**

`ansible-playbook` 실행 → **site.yml**이 전체 작업 시작 → **Playbook**이 **Role** 호출 → **Role**의 **Task(main.yml)** 실행 → 서버 구성 및 서비스 자동 구축 완료


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
---
**playbook/site.yml 코드 예시**

```
전체 인프라 구축의 실행 진입점으로, 서버 환경에 필요한 Role을 순서대로 호출하여
Docker, Kubernetes, Harbor 등의 구성 작업을 자동 실행합니다.
DB는 사전에 구성해서 제외합니다.

---
- name: Server Init
  hosts: all
  become: yes
  roles:
    - docker


- name: Kubernetes Install
  hosts:
    - k8s_master
    - k8s_worker
  become: yes
  roles:
    - kubernetes


- name: Harbor Setup
  hosts: harbor
  become: yes
  roles:
    - harbor
```
---

**roles 코드 예시**
```
# roles/kubernetes/tasks/main.yml
Kubernetes Role의 main.yml은 설치 작업을 먼저 수행한 후,
Inventory 그룹 정보를 기준으로 Master Node와 Worker Node 작업을 분리하여 실행합니다.

---
# Kubernetes 설치 작업 실행
- import_tasks: install.yml


# Master Node 설정
- import_tasks: master.yml
  when: inventory_hostname in groups['k8s-master']


# Worker Node 설정
- import_tasks: worker.yml
  when: inventory_hostname in groups['k8s-worker']
```
---

```
# roles/harbor/tasks/main.yml

Harbor Role은 필수 패키지 설치 후 설정 파일을 배포하여 Private Container Registry 환경을 자동 구성합니다.

---
# Harbor 설치에 필요한 패키지 설치
- name: Install harbor dependencies
  apt:
    name:
      - curl
      - wget
      - docker-compose-plugin
    state: present


# Harbor 설치 디렉토리 생성
- name: Create harbor directory
  file:
    path: /opt/harbor
    state: directory


# Harbor 설정 파일 배포
- name: Copy harbor config
  copy:
    src: harbor.yml
    dest: /opt/harbor/harbor.yml


```
---

```
# roles/docker/tasks/main.yml

Docker Role은 서버에 Docker를 설치하고 서비스 실행 상태를 유지하도록 자동 구성합니다.

---
# Docker 패키지 설치
- name: Install Docker
  apt:
    name:
      - docker.io
    state: present


# Docker 서비스 실행 및 부팅 자동 시작 설정
- name: Start Docker Service
  service:
    name: docker
    state: started
    enabled: yes


```
---

```
# roles/database/tasks/main.yml

Database Role은 서비스별 Task를 분리하여 PostgreSQL, Redis, Kafka 환경을 자동 구성합니다.

---
# PostgreSQL 설치 및 설정
- name: Install PostgreSQL
  import_tasks: postgres.yml


# Redis 설치 및 설정
- name: Install Redis
  import_tasks: redis.yml


# Kafka 설치 및 설정
- name: Install Kafka
  import_tasks: kafka.yml
```
---

**inbentory 코드 예시**

```
d
```

# CI/CD Pipeline

```
GitLab CI/CD 동작 과정
GitLab CI/CD Pipeline을 통해 코드 변경 사항을 감지하고 자동으로 빌드 및 배포 과정을 수행합니다.

GitLab Repository에 코드 Push
        ↓
GitLab CI Pipeline 실행
        ↓
Docker Image Build
        ↓
Harbor Private Registry Push
        ↓
Kubernetes Cluster에 Image 배포
        ↓
kubectl 명령으로 Pod 업데이트
```


# 서비스 구성
(DB / Redis / Kafka 등)

# 트러블슈팅 / 장애 해결

# 결과 및 성과
웹 만든거 구현 화면 여기에 넣기
# 느낀점
