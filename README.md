# GitLab CI/CD와 Ansible을 활용한 자동화 창고관리시스템 구축

# 프로젝트 목적

본 프로젝트는 GitLab CI/CD와 Ansible을 활용한 자동화 창고관리시스템(WMS) 구축 프로젝트다.

GitLab Pipeline을 통해 소스 코드 관리부터 빌드, 테스트, 배포까지의 CI/CD 프로세스를 자동화하고, Ansible을 이용해 서버 환경 구성 및 애플리케이션 배포를 코드 기반으로 관리한다.

이를 통해 반복적인 수동 작업을 최소화하고, 안정적이고 일관된 운영 환경을 구축하여 창고관리시스템의 배포 효율성과 유지보수성을 향상시키는 것을 목표로 한다.

# 구현 범위

<img width="1000" height="600" alt="image" src="https://github.com/user-attachments/assets/f53cd3ee-d943-4cd0-93fa-f7cc3c5ae210" />
<br>
<br>

여러 대의 서버를 하나로 묶어 관리하는 환경 구성 , 파이썬 프로그램, 대용량 데이터 처리 장치(Kafka)를 연결해 만든 창고 관리 시스템


**핵심 트래픽 및 데이터 흐름**
- 사용자 진입: 직원이 PC나 모바일 웹 화면으로 접속하면, NGINX라는 프로그램이 가장 먼저 요청을 받아 안전하게 내부 서버들로 나누어 보낸다.

- 기능별 서버 분리: 입고, 출고, 재고(위치/이동), QR 스캔 기능이 각각 별도의 독립된 방(컨테이너)에서 실행한다. 하나의 기능에 문제가 생기거나 사용자가 몰려도 다른 기능은 영향을 받지 않는다.

- 데이터 저장 방식: 중요한 마스터 정보는 메인 저장소(PostgreSQL)에 안전하게 저장하고, 자주 쓰는 정보는 초고속 임시 저장소(Redis)에 두어 화면이 빠르게 뜨도록 한다. 현장에서 QR 코드를 대량으로 찍을 때 발생하는 과부하는 중간 완충 장치(Kafka)가 받아주어 서버가 다운되는 것을 막는다.
<br>

**프로그램 개발 및 자동 설치 과정**

- 자동 배포: 개발자가 수정된 코드를 깃랩(GitLab)에 올리면, 시스템이 알아서 이를 포장하여 안전한 보관소(Harbor)에 저장한 뒤, 실제 운영 중인 서버에 사용자가 눈치채지 못하도록 멈춤 없이 자동으로 교체 및 설치한다.

# 담당 역할
앤서블을 이용하여 서버 자동 구축

[Ansible 작업 기록](https://app.notion.com/p/Ansible-390a48359dbb80f78483d187e0ede9c0?source=copy_link)

```
Role 구성 → Inventory 작성 → Playbook 작성 → site.yml 작성(전체 실행) → ansible-playbook 실행 → 서버 자동 구축 완료
```
<br>
<br>

깃랩 CI/CD 파이프라인 구축

[Gitlab 작업 기록](https://www.notion.so/Ansible-390a48359dbb80f78483d187e0ede9c0?source=copy_link)

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



# Ansible 서버 자동화 구성



Ansible은 Playbook을 실행하여 여러 Role을 순차적으로 호출하고, 각 Role이 담당하는 서버 구성을 자동으로 수행하는 구조로 구성되어 있다.

**Ansible 동작 과정**

`ansible-playbook` 실행 → **site.yml**이 전체 작업 시작 → **Playbook**이 **Role** 호출 → **Role**의 **Task(main.yml)** 실행 → 서버 구성 및 서비스 자동 구축 완료


```
├── ansible.cfg
├── group_vars
│   └── all.yml
├── inventory
│   └── hosts.ini
├── playbooks
│   ├── deploy.yml
│   ├── gitlab-install.yml
│   ├── init-server.yml
│   ├── k8s-master-init.yml
│   ├── k8s-worker-join.yml
│   ├── kubernetes-install.yml
│   ├── reset.yml
│   └── site.yml
└── roles
    ├── common
    │       └── main.yml
    ├── database
    │       ├── kafka.yml
    │       ├── main.yml
    │       ├── postgres.yml
    │       └── redis.yml
    ├── gitlab  
    │  └── main.yml
    ├── gitlab-runner
    │   └── main.yml
    ├── harbor
    │       └── main.yml
    │       └── harbor.yml.j2
    ├── k8s-master
    │       └── main.yml
    └── k8s-worker
            └── main.yml

```
---
**playbook/site.yml 코드 예시**

```
전체 인프라 구축의 실행 진입점으로, 서버 환경에 필요한 Role을 순서대로 호출하여
Kubernetes, Harbor 등의 구성 작업을 자동 실행한다.
DB는 사전에 구성해서 제외한다.

- name: Harbor 서버 구성 
  hosts: harbor
  become: true
  roles:
    - harbor

- name: K8s 노드 공통 설정 (마스터+워커, containerd/swap/sysctl)
  hosts: k8s_cluster
  become: true
  roles:
    - common

- name: K8s 마스터 노드 구성 (192.168.159.120)
  hosts: master
  become: true
  roles:
    - k8s-master

- name: K8s 워커 노드 구성 및 클러스터 join 
  hosts: worker
  become: true
  roles:
    - k8s-worker

```
---

**roles 코드 예시**

```
# roles/k8s-master/main.yml
Master 구성하는 main.yml 파일
---
# 1. K8s 컴포넌트 설치 및 버전 고정
- name: Install K8s packages (kubeadm, kubelet, kubectl)
  apt: name: [kubelet, kubeadm, kubectl]

# 2. 기존 클러스터 흔적 및 네트워크 초기화 (충돌 방지)
- name: Reset existing cluster configurations
  command: kubeadm reset -f

# 3. 마스터 노드 초기화 및 클러스터 생성
- name: Initialize K8s master node
  command: kubeadm init --apiserver-advertise-address={{ master_ip }}

# 4. 일반 사용자용 kubeconfig 설정
- name: Configure kubeconfig for ansible_user
  copy: src: /etc/kubernetes/admin.conf dest: ~/.kube/config

# 5. Pod 네트워크(Calico CNI) 배포
- name: Deploy Calico CNI network plugin
  command: kubectl apply -f {{ calico_manifest_url }}

# 6. Worker 노드 추가용 Join 명령어 생성 및 저장
- name: Generate and save worker node join command
  command: kubeadm token create --print-join-command

# 7. 사설 Harbor 인증서 등록 디렉토리 생성
- name: Create Harbor CA certificate directory
  file: path: /usr/local/share/ca-certificates/harbor

```
---
```
# roles/k8s-master/main.yml
worker node 구성하는 main.yml 파일
---
# 1. K8s 컴포넌트 설치 및 버전 고정
- name: Install K8s packages (kubeadm, kubelet, kubectl)
  apt: name: [kubelet, kubeadm, kubectl]

# 2. 기존 설정 및 네트워크 초기화 (충돌 방지)
- name: Reset existing worker configurations
  command: kubeadm reset -f

# 3. 로컬에 저장된 Join 명령어(토큰) 로드
- name: Load saved join command from control node
  local_action: slurp src=/tmp/k8s_join_command.sh

# 4. 마스터 노드 클러스터에 합류 (Join)
- name: Execute join command to join cluster
  command: "{{ join_command_file.content | b64decode }}"

```
---
```
# roles/harbor/tasks/main.yml

Harbor Role은 필수 패키지 설치 후 설정 파일을 배포하여 Private Container Registry 환경을 자동 구성한다.

# 1. 호스트 환경 설정 및 IP 등록
- name: Clear machine-id & Set hostname
  hostname: name: harbor-server

- name: Configure /etc/hosts file
  blockinfile: path: /etc/hosts block: "{{ master_ip }} k8s-master ..."

# 2. Docker 및 docker-compose 설치
- name: Install Docker and Compose plugin
  apt: name: [docker-ce, docker-ce-cli, containerd.io, docker-compose-plugin]

- name: Enable Docker service & Add user to docker group
  systemd: name: docker enabled: true

# 3. Harbor용 자체서명 TLS 인증서(OpenSSL) 생성
- name: Create certificate directory & Generate Root CA
  command: openssl req -x509 -new -nodes -days 3650 ...

- name: Generate and Sign Harbor Server Certificate (with SAN IP)
  command: openssl x509 -req -extfile v3.ext -CA ca.crt ...

# 4. Harbor 설치 파일 다운로드 및 압축 해제
- name: Download Harbor offline installer and extract
  unarchive: src: /tmp/harbor-offline-installer.tgz dest: /opt

# 5. 설정 파일 배포 (Jinja2 템플릿 사용)
- name: Generate harbor.yml configuration file
  template: src: harbor.yml.j2 dest: "{{ harbor_install_dir }}/harbor.yml"

# 6. Harbor 설치 스크립트 실행
- name: Execute Harbor install script
  command: "./install.sh" chdir="{{ harbor_install_dir }}"

```
---

```
# roles/gitlab-runner/tasks/main.yml

gitlab-runner Role은 배포의 파이프 라인을 형성한다.
---
# tasks file for roles/gitlab-runner
- name: Install GitLab Runner dependencies
  apt:
    name:
      - curl
      - ca-certificates
      - gnupg
    state: present
    update_cache: yes


- name: Add GitLab Runner repository script
  shell: |
    curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | bash
  args:
    creates: /etc/apt/sources.list.d/runner_gitlab-runner.list


- name: Install GitLab Runner
  apt:
    name: gitlab-runner
    state: present
    update_cache: yes


- name: Enable GitLab Runner service
  systemd:
    name: gitlab-runner
    enabled: yes
    state: started
```
---

```
# roles/gitlab/tasks/main.yml

Ansible 을 이용하여 CI/CD의 핵심인 GitLab 서버를 구축한다.
---
- name: Load gitlab variables
  include_vars:
    file: ../../../group_vars/all.yml


- name: Install GitLab dependencies
  apt:
    name:
      - curl
      - openssh-server
      - ca-certificates
      - tzdata
    state: present
    update_cache: yes


- name: Add GitLab repository
  shell: |
    curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | bash


- name: Install GitLab
  apt:
    name: gitlab-ee
    state: present


- name: Configure GitLab external url
  lineinfile:
    path: /etc/gitlab/gitlab.rb
    regexp: "^external_url"
    line: "external_url 'http://10.1.201.127'"
  notify:
    - Reconfigure GitLab

```
---
**inbentory 코드 예시**
네트워크 구성 끝나면 바로 작성 할 것

```
d
```

# CI/CD Pipeline

```
GitLab CI/CD 동작 과정
GitLab CI/CD Pipeline을 통해 코드 변경 사항을 감지하고 자동으로 빌드 및 배포 과정을 수행한다.

Git push
  ↓
GitLab CI
  ↓
Docker build
  ↓
Harbor push (저장소)
  ↓
K8s deploy
```
<br>
위와 같은 파이프 라인을 구성하기 위해 .gitlab-ci.yml 파일을 작성하여 프로젝트의 웹을 배포 한다.
<br>

```
.gitlab-ci.yml 
stages:
  - test
  - build

variables:
  IMAGE_NAME: test-app
  TAG: $CI_COMMIT_SHORT_SHA

# 1. CI 정상 확인용
test:
  stage: test
  tags:
    - docker
  script:
    - echo "CI PIPELINE WORKING"
    - echo "Runner OK"
    - uname -a
    - echo "Network independent mode"

# 2. 가짜 build (외부 없음)
build:
  stage: build
  tags:
    - docker
  script:
    - echo "BUILD STAGE START"
    - echo "No Docker build yet (network not ready)"
    - echo "Simulating build process"
    - mkdir -p dist
    - echo "dummy artifact" > dist/app.txt
  artifacts:
    paths:
      - dist/
```

# 서비스 구성
(DB / Redis / Kafka 등)
쿠버네티스 구성하는 사람꺼 인용하기

# 트러블슈팅 / 장애 해결

# 결과 및 성과
웹 만든거 구현 화면 여기에 넣기
# 느낀점
