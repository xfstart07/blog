---
title: 搭建GitLab CI
date: 2018-06-10 11:41:55
categories:
  - Devops
tags:
  - Devops
  - gitlab
  - CI
---

获取 gitlab-ci Token

用管理员账户登录，在路径 `/admin/runner` 获得Token和URL。

选择 docker 作为 runner 的引擎，需要安装 docker。

### 通过自动安装脚本安装 Docker

```
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```

## 使用 gitlab job

编写 .gitlab-ci.yml

```
image: golang:1.10.2
stages:
  - test
  - build

before_script:
  - echo $PATH
  - cd $GOPATH/src
  - git clone http://repo_path.git
  - cd $GOPATH/src/reponame

test-adserver:
  stage: test
  script:
    - cd $GOPATH/src/reponame
    - git checkout testbranch
    - go vet
    - go test -v -cover ./...

build-adserver:
  stage: build
  script:
    - cd $GOPATH/src/reponame
    - git checkout buildbranch
    - go build
```

配置文件说明：创建 2个 stage，分别是测试和构建，在运行之前通过`before_script`来初始化环境。
