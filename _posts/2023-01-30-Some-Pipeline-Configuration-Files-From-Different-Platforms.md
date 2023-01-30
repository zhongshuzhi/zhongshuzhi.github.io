---
title: 一些平台的流水线的代码化配置实例
date: 2023-01-30 11:18:23
tags: Pipeline
---

# 一些平台的流水线的代码化配置实例

因为最近在做流水线配置代码化的相关功能，因此对参考的一些流水线代码化配置做记录，以便后续参考。

## GitLab CI/CD

Reference: <https://meigit.readthedocs.io/en/latest/gitlab_ci_.gitlab-ci.yml_detail.html>

```yaml
# This file is a template, and might need editing before it works on your project.
# see https://docs.gitlab.com/ce/ci/yaml/README.html for all available options

# 定义全局变量
# Variables of different types can take precedence over other variables, depending on where they are defined.

#  The order of precedence for variables is (from highest to lowest):
#    Trigger variables or scheduled pipeline variables.
#    Project-level variables or protected variables.
#    Group-level variables or protected variables.
#    YAML-defined job-level variables.
#    YAML-defined global variables.
#    Deployment variables.
#    Predefined environment variables.

variables:
  # 数据库信息
  SQLALCHEMY_DATABASE_URI: 'mysql+pymysql://root:root@localhost:3306/bluelog?charset=utf8mb4'
  # 不发送警告通知
  SQLALCHEMY_TRACK_MODIFICATIONS: "False"
  # 显示执行SQL
  SQLALCHEMY_ECHO: "True"

# 流水线开始执行之前的前置脚本
before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

# 流水线执行完成之后的后置脚本
after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

stages:
  - build
  - code_check
  - test
  - deploy
  - review_app
  - stop_review_app

build1:
  stage: build
  variables: #定义局部变量
    # 数据库信息
    SQLALCHEMY_DATABASE_URI: 'mysql+pymysql://root:123456@localhost:3306/bluelog?charset=utf8mb4'
    # 不显示执行SQL
    SQLALCHEMY_ECHO: "False"
  before_script: #Job执行前的脚本
    - echo "Before script in build stage that overwrited the globally defined before_script"
    - echo "Install cloc:A tool to count lines of code in various languages from a given directory."
    - yum install cloc -y
  after_script:
    - echo "After script in build stage that overwrited the globally defined after_script"
    - cloc --version
    # cloc .
  only:
    - /^issue-.*$/
  except:
    - master
  script:
    - echo "Do your build here"
    - cloc --version
    # - cloc .
  artifacts:
    when: always #无论成功失败都上传附件
    paths:
      - binaries/
      - .config
  tags:
    - bluelog

find Bugs:
  stage: code_check
  only: #仅分支名为issue-开头的，以及tag和trigger触发时，才会执行流水线
    - /^issue-.*$/
    - triggers
  except:
    - branches
  script:
    - echo "Use Flake8 to check python code"
    - pip install flake8
    - flake8 --version
    # - flake8 .
  allow_failure: true # 允许失败后继续
  tags:
    - bluelog

test1:
  stage: test
  only:
    - /^issue-.*$/
  except:
    - /issue-pylint/
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
  tags:
    - bluelog
  when: always

test2:
  stage: test
  only:
    - /^issue-.*$/
  except:
    - /Issue-flake8/
  script:
    - echo "Do another parallel test here"
    - echo "For example run a lint test"
  tags:
    - bluelog #指定这个Job的执行器
  when: manual
  retry: 
    max: 2 #失败后重试2次
    when: runner_system_failure #仅当runner系统故障时重试

deploy1:
  stage: deploy
  only:
    - /^issue-.*$/
  except:
    - /severe-issues/
  script:
    - echo "Do your deploy here"
  environment:
    name: production
    url: https://prod.example.com
  tags:
    - bluelog
  dependencies: []

# 在下面的示例中，设置 review_app 作业用于部署代码到 review 评审环境中，同时在 on_stop 中指定了 stop_review_app 作业。一旦 review_app 作业成功执行，就会触发 when 关键字定义的 stop_review_app 作业。通过设置为 manual 手动，需要在GitLab WEB界面点击来允许 manual action 。

review_app:
  stage: deploy
  script: make deploy-app
  environment:
    name: review
    on_stop: stop_review_app

stop_review_app:
  stage: deploy
  script: make delete-app
  when: manual
  environment:
    name: review
    action: stop
```

------

## Gitee Go

Reference: <https://gitee.com/help/articles/4292#article-header0>

```yaml
# ========================================================
# Gitee Go 流水线配置样例文件
# 功能：实现一个 Maven 命令行工程初始化并构建
# ========================================================
name: gitee-go-maven                   # 流水线唯一标识，定义一个唯一 ID 标识为 gitee-go-maven，名称为 “Maven-流水线示例” 的流水线
displayName: 'Maven-流水线示例'          # 流水线名字
triggers:                              # 流水线触发器配置，支持通过 push 事件触发构建
  push:                                # 通过 push方式触发
    - matchType: PRECISE               # matchType
      branch: master                   # 触发分支名（值为字符串），目前为当前所在分支名，暂不支持跨分支触发
commitMessage: ''                      # 通过匹配当前提交的 CommitMessage 决定是否执行流水线
stages:                                # 构建阶段配置
  - stage:                             # 单个构建阶段
      name: maven-build-stage          # 构建阶段唯一标识
      displayName: 'Maven Stage'       # 构建阶段名称
      failFast: false                  # 允许快速失败，即当 Stage 中有任务失败时，直接结束整个 Stage
      steps:                           # 构建步骤配置
        - step: mavenbuild@1           # 构建步骤的任务类型(枚举类型)，用于决定使用什么构建环境。目前不支持自定义。当前示例为采用 Maven 编译环境
          name: maven-build            # 构建步骤唯一标识，当前示例中定义了一个标识为 maven-build 的构建步骤
          displayName: 'Maven Step'    # 构建步骤名称当前示例中定义了一个名为 “Maven Step” 的构建步骤
          inputs:                      # 构建输入参数设定
            mavenPomFile: 'pom.xml'    # pom文件位置，非必填项
            jdkVersion: 8              # 语言版本，指定 JDK 环境版本为 1.8
            mavenVersion: 3.6          # 工具版本，指定 Maven 环境版本为 3.6
            goals: |                   # 构建脚本，当前示例中使用 Maven 命令初始化、构建一个 Maven 工程并执行输出内容
              mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DinteractiveMode=false -DarchetypeCatalog=internal -DgroupId=com.gitee.go.maven -DartifactId=helloworld -T20
              cd helloworld 
              mvn clean
              mvn compile
              mvn test-compile
              mvn package
              java -cp target/helloworld-1.0-SNAPSHOT.jar com.gitee.go.maven.App
```

------

## Jenkins


**待补充**
------

## CDS

Reference:

<https://github.com/ovh/cds>

<https://ovh.github.io/cds/docs/concepts/files/>

- Pipeline configuration file

```yaml
version: v1.0
name: build

parameters:
  param_name:
    type: string
    default: default_value
    
stages:
- Compile
- Package


jobs:
- job: Build UI
  stage: Compile
  steps:
  - gitClone:
      branch: '{{.git.branch}}'
      commit: '{{.git.hash}}'
      directory: cds
      url: '{{.git.url}}'
  - script:
    - echo {{.cds.pip.param_name}}  
    - cd cds/ui
    - npm set registry https://registry.npmjs.org
    - npm install
    - ng build -prod --aot
    - tar cfz ui.tar.gz dist
  - artifactUpload:
      path: cds/ui/ui.tar.gz
      tag: '{{.cds.version}}'
  requirements:
  - binary: git
  - memory: "1024"
  - model: Node8.9.1

- job: Test UI
  stage: Compile
  enabled: false
  requirements:
  - binary: git
  - memory: "1024"
  - model: Node8.9.1

  steps:
  - gitClone:
      branch: '{{.cdsbuildgitbranch}}'
      commit: '{{.git.hash}}'
      directory: cds
      password: ""
      privateKey: ""
      url: '{{.cds.app.repo}}'
      user: ""
  - script:
    - export CHROME_BIN=chromium
    - npm set registry https://registry.npmjs.org
    - cd cds/ui
    - npm install
    - npm test

  - jUnitReport: ./cds/ui/tests/**/results.xml

- job: Package UI
  stage: Package
  requirements:
  - binary: docker
  steps:
  - artifactDownload:
      path: .

  - CDS_DockerPackage:
      dockerfileDirectory: .
      imageName: ovh/cds-ui
      imageTag: '{{.cds.version}}'
```

- Workflow configuration file

A CDS workflow file only contains the description of pipelines orchestration, hooks, run conditions, etc.. Consider the following workflow which implements a basic two-stage workflow:

```yaml
name: my-workflow
workflow:
  build:
    pipeline: build
    application: my-application
  deploy:
    depends_on:
    - build
    when:
    - success
    pipeline: deploy
    application: my-application
    environment: my-production
    one_at_a_time: true
hooks:
  build:
  - type: RepositoryWebHook
integrations:
  my-artifactory-integration-name:
    type: artifact_manager
notifications:
- type: email
  pipelines:
  - deploy
  settings:
    on_success: never
    recipients:
    - me@foo.bar
retention_policy: return run_days_before < 7
```

There are two major things to understand: `workflow` and `hooks`. A workflow is a kind of graph starting from a root pipeline, and other pipelines with dependencies. In this example, the `deploy` pipeline will be triggered after the `build` pipeline

- Action configuration file

Hello World Action

```yml
version: v1.0
name: CDS_HelloWorld
description: Hello World Action
steps:
- name: Initialization
  script:
  - echo "Hello World"
```

With a real action `CDS_SonarScanner`: this action contains parameters with default values and some of them are `advanced` parameters. Two plugins are also used in the steps: `plugin-download` and `plugin-archive`

```yaml
version: v1.0
name: CDS_SonarScanner
description: Run Sonar analysis. You must have a file sonar-project.properties in
  your source directory.
parameters:
  sonar-project.properties:
    type: text
    default: |-
      sonar.projectKey={{.cds.application}}
      sonar.projectName={{.cds.application}}
      sonar.projectVersion={{.git.hash}}
      sonar.sources=.
      sonar.exclusions=**/*_test.go,**/vendor/**
      sonar.tests=.
      sonar.test.inclusions=**/*_test.go
      sonar.test.exclusions=**/vendor/**      
    description: sonar-project.properties file
  sonarBranch:
    type: string
    default: '{{.git.branch}}'
    description: The Sonar branch (e.g. master)
  sonarDownloadURL:
    type: string
    default: https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-{{.sonarVersion}}-linux.zip
    description: The download URL of Sonar CLI
    advanced: true
  sonarPassword:
    type: string
    default: '{{.cds.proj.sonarPassword}}'
    description: The Sonar server's password
    advanced: true
  sonarURL:
    type: string
    default: '{{.cds.proj.sonarURL}}'
    description: The URL of the Sonar server
    advanced: true
  sonarUsername:
    type: string
    default: '{{.cds.proj.sonarUsername}}'
    description: The Sonar server's username
    advanced: true
  sonarVersion:
    type: string
    default: 3.2.0.1227
    description: SonarScanner's version to use
    advanced: true
  workspace:
    type: string
    default: '{{.cds.workspace}}'
    description: The directory where your project is (e.g. /go/src/github.com/ovh/cds)
requirements:
- binary: bash
- plugin: plugin-archive
- plugin: plugin-download
steps:
- name: Initialization
  script:
  - '#!/bin/bash'
  - set -x
  - '# Installation'
  - mkdir -p {{.workspace}}/opt
- plugin-download:
    filepath: '{{.workspace}}/opt/sonar-scanner-cli-{{.sonarVersion}}-linux.zip'
    url: '{{.sonarDownloadURL}}'
- plugin-archive:
    action: uncompress
    destination: '{{.workspace}}/opt/'
    source: '{{.workspace}}/opt/sonar-scanner-cli-{{.sonarVersion}}-linux.zip'
- script:
  - '#!/bin/bash'
  - set -x
  - ""
  - '# Installation'
  - ln -s {{.workspace}}/opt/sonar-scanner-{{.sonarVersion}}-linux {{.workspace}}/opt/sonar
  - export PATH="${PATH}:{{.workspace}}/opt/sonar/bin"
  - ""
  - '# Runtime'
  - export SONAR_SCANNER_OPTS="-Xmx1024m"
  - cd {{.workspace}}
  - cat <<EOF > sonar-project.properties
  - '{{.sonar-project.properties}}'
  - EOF
  - ""
  - sonar-scanner -Dsonar.host.url={{.sonarURL}} -Dsonar.login={{.sonarUsername}}
    -Dsonar.password={{.sonarPassword}} -Dsonar.branch={{.sonarBranch}} -Dsonar.scm.disabled=true
```

**中文注释待补充**
