# 12. Lab 09: CI/CD ด้วย GitLab CE และ GitLab Runner

## วัตถุประสงค์

- ติดตั้ง GitLab CE แบบ container
- ตั้ง GitLab Runner
- เขียน `.gitlab-ci.yml`
- build และ push image เข้า registry

## ติดตั้ง GitLab CE บน devops-control

```bash
mkdir -p ~/gitlab/{config,logs,data}
docker run -d \
  --hostname devops-control \
  --name gitlab \
  --restart always \
  -p 8081:80 \
  -p 2222:22 \
  -v ~/gitlab/config:/etc/gitlab \
  -v ~/gitlab/logs:/var/log/gitlab \
  -v ~/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

ดู password root:

```bash
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

เข้าใช้งาน:

```text
http://devops-control:8081
```

## ติดตั้ง GitLab Runner

```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

Register runner จาก GitLab UI แล้วเลือก executor เป็น docker

## ตัวอย่าง .gitlab-ci.yml

```yaml
stages:
  - test
  - build
  - deploy

variables:
  IMAGE_NAME: "devops-control:5000/simple-api"
  IMAGE_TAG: "$CI_COMMIT_SHORT_SHA"

test:
  stage: test
  image: node:20-alpine
  script:
    - npm ci
    - node -e "console.log('test passed')"

build:
  stage: build
  image: docker:latest
  script:
    - docker build -t $IMAGE_NAME:$IMAGE_TAG .
    - docker push $IMAGE_NAME:$IMAGE_TAG

deploy_dev:
  stage: deploy
  script:
    - echo "Deploy to dev server here"
  only:
    - main
```

## การทดสอบ

- Push code เข้า GitLab
- Pipeline ต้องรันครบ stage
- Registry มี image tag ใหม่

## การตรวจสอบ

```bash
curl http://devops-control:5000/v2/_catalog
```

## ข้อสรุป

CI/CD ลดงาน manual และทำให้ทุก release มีประวัติ ตรวจสอบได้ และทำซ้ำได้

---
