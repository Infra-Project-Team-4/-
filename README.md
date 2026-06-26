**Ubuntu에서 자기 fork clone**

```jsx
git clone git@github.com:yeonxp/hi.git
cd hi
```

**원본 organization repo 연결**

```jsx
git remote add upstream git@github.com:Infra-Project-Team-4/hi.git

git remote -v
```

나는 첨에 upload test 할 떄 이미 추가 된 상태라 수정

```jsx
git remote set-url origin git@github.com:yeonXP/hi.git

git remote add upstream git@github.com:Infra-Project-Team-4/hi.git

git remote -v
```

```
origin = 내 저장소 (내 GitHub 원격 저장소)<br>
upstream = 원본 저장소 (타인의 저장소)
```

#### 작업 시 명령어

**원본 최신 내용 가져오기**

```jsx
git checkout main
git pull upstream main
```

**내 fork에도 최신 내용 반영**

```jsx
git push origin main
```

**작업 브랜치 만들기**

```jsx
git checkout -b feature/service-health-api
```

**파일 수정 후 commit**

```jsx
git status
git add .
git commit -m "add health check api"
```

**내 fork에 push**

```jsx
git push origin feature/service-health-api
```
