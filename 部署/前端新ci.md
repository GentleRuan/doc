[TOC]

# 一、新建/修改ci

## Vue静态项目

新建.gitlab-ci.yml文件，内容在最下面，push到相应的feature分支上

ps:如果vue项目的名字和nodejs的项目名字不一样ci中定义变量做如下修改：

```
STATIC_PROJECT_NAME: "nodejs项目目录"
```


## NodeJs项目

修改.gitlab-ci.yml文件，内容在最下面，push到相应的feature分支上

ps:如果静态文件不目录是不dist目录，app.js里修改如下(dist目录做为静态文件映射出去)：
```
app.use(express.static(path.join(__dirname, 'dist')));
```


# 二、项目发布

在git上发merge request（先发布VUE项目、再发布NodeJs项目）

## 测试环境发布

```
graph LR
feature -.merge .-> A[test]
A-->B[自动发布]
```

## 正式环境发布

```
graph LR
test -. merge .-> A[master]
A -.打tag .-> B[v1.0.0]
B --> C[手动发布tag]
```

## 发布分支说明

分支|说明
---|---|---
test|测试环境
tag|正式环境


# 资料
## 之前的发布流程

```
graph LR
VUE项目-.本地打包生成.->B[dist目录]
B -. 复制dist放在.-> C[NodeJs]
C -. 生成 .-> D[Docker镜像]
D -.-> 发布
```

## 新的流程


```
graph LR
VUE项目 -. 通过Ci打包生成 .-> A[dist目录]
A -.压缩dist生成 .-> A1[dist.tar.gz]
A1 -. 上传 .-> A2[OSS]

```

```
graph LR
NodeJs项目 -. 通过ci拉取 .-> A[OSS上dist.tar.gz文件]
A -. 解压生成dist放在 .-> A1[根目录] 
A1 -. 生成 .-> D[Docker镜像]
D -.-> 发布

```




## Vue静态项目.gitlab-ci.yml

```
variables:
  PROJECT_GROUP: ${CI_PROJECT_NAMESPACE}
  PROJECT_NAME: ${CI_PROJECT_NAME}
  # docker镜像
  NODEJS_IMAGE: "docker.source3g.com:5000/library/nodejs:12.13.1"
  # sonar访问地址
  SONAR_HOST: "https://sonar.wangxiaobao.com"
  # 上传文件地址
  UP_URL: "http://api-i-prod-e.source3g.com/alioss/upload"
  # nodejs项目目录
  STATIC_PROJECT_NAME: ${CI_PROJECT_NAME}
  # 压缩完后的文件名
  STATIC_FILE_NAME: "dist.tar.gz"
  # 要压缩的文件夹
  STATIC_CATALOG: "dist"
#定义CI阶段
stages:
  - sonar
  - build


sonar:default:
  image: ${DOCKER_REGISTRY}/env/sonarscanner:v1.02
  stage: sonar
  script:
    - /sonar-scanner/bin/sonar-scanner 
        -Dsonar.projectKey=${PROJECT_GROUP}.${PROJECT_NAME}.${CI_COMMIT_REF_NAME} 
        -Dsonar.sources=/builds/${PROJECT_GROUP}/${PROJECT_NAME} 
        -Dsonar.projectName=${PROJECT_GROUP}.${PROJECT_NAME}.${CI_COMMIT_REF_NAME} 
        -Dsonar.host.url=${SONAR_HOST} 
        -Dsonar.login=f6008364db3b52711b641ab87d7981445f760545 
        -Dsonar.gitlab.project_id=${CI_PROJECT_PATH} 
        -Dsonar.gitlab.commit_sha=${CI_COMMIT_SHA} 
        -Dsonar.gitlab.ref_name=${CI_COMMIT_REF_NAME}
  except:
    - test
    - tags
sonar:test:
  image: ${DOCKER_REGISTRY}/env/sonarscanner:v1.02
  stage: sonar
  script:
    - /sonar-scanner/bin/sonar-scanner 
        -Dsonar.projectKey=${PROJECT_GROUP}.${PROJECT_NAME}.test 
        -Dsonar.sources=/builds/${PROJECT_GROUP}/${PROJECT_NAME} 
        -Dsonar.projectName=${PROJECT_GROUP}.${PROJECT_NAME}.test 
        -Dsonar.host.url=${SONAR_HOST}
        -Dsonar.login=f6008364db3b52711b641ab87d7981445f760545 
        -Dsonar.gitlab.project_id=${CI_PROJECT_PATH} 
        -Dsonar.gitlab.commit_sha=${CI_COMMIT_SHA} 
        -Dsonar.gitlab.ref_name=${CI_COMMIT_REF_NAME} | tee log.txt
    - grep -q 'desc=SonarQube reported QualityGate is ok' log.txt && exit 0 || exit -1
  only:
    - test
sonar:prod:
  image: ${DOCKER_REGISTRY}/env/sonarscanner:v1.02
  stage: sonar
  script:
    - /sonar-scanner/bin/sonar-scanner -Dsonar.projectKey=${PROJECT_GROUP}.${PROJECT_NAME}.prod 
        -Dsonar.sources=/builds/${PROJECT_GROUP}/${PROJECT_NAME} 
        -Dsonar.projectName=${PROJECT_GROUP}.${PROJECT_NAME}.prod 
        -Dsonar.host.url=${SONAR_HOST}
        -Dsonar.login=f6008364db3b52711b641ab87d7981445f760545 
        -Dsonar.gitlab.project_id=${CI_PROJECT_PATH} 
        -Dsonar.gitlab.commit_sha=${CI_COMMIT_SHA} 
        -Dsonar.gitlab.ref_name=${CI_COMMIT_REF_NAME} | tee log.txt
    - grep -q 'desc=SonarQube reported QualityGate is ok' log.txt && exit 0 || exit -1
  only:
    - tags 


# 测试环境
build:test:
  image: ${NODEJS_IMAGE}
  stage: build
  script:
    - pwd
    - cnpm install
    - npm run build    
    - tar -czvf ${STATIC_FILE_NAME} ${STATIC_CATALOG}
    - curl ${UP_URL} -X PUT -F "file=@${STATIC_FILE_NAME}" -F "name=/ftd-builds/test/${STATIC_PROJECT_NAME}/${STATIC_FILE_NAME}"
  only:
    - test
  # artifacts:
  #   paths:
  #     - dist/

 # 正式环境
build:prod:
  image: ${NODEJS_IMAGE}
  stage: build
  script:
    - ls -la
    - cnpm install
    - npm run build    
    - tar -czvf ${STATIC_FILE_NAME} ${STATIC_CATALOG}
    - curl ${UP_URL} -X PUT -F "file=@${STATIC_FILE_NAME}" -F "name=/ftd-builds/prod/${STATIC_PROJECT_NAME}/${STATIC_FILE_NAME}"
  # artifacts:
  #   paths:
  #     - dist/
  only:
    - tags
```

## NodeJs项目.gitlab-ci.yml


```
variables:
  PROJECT_GROUP: ${CI_PROJECT_NAMESPACE}
  PROJECT_NAME: ${CI_PROJECT_NAME}
  STATIC_PROJECT_NAME: ${CI_PROJECT_NAME}
  # sonar访问地址
  SONAR_HOST: "https://sonar.wangxiaobao.com"
  # oss内网访问地址
  STATIC_PATH: "http://gstf-front-end.oss-cn-beijing-internal.aliyuncs.com/ftd-builds"
  # 静态资源文件名
  STATIC_FILE_NAME: "dist.tar.gz"
  # 静态资源解压到的目录
  STATIC_CATALOG: "dist"
  ENV_DEV: "dev"
  ENV_TEST: "test"
#定义CI阶段
stages:
  - sonar
  - package
  - deploy

sonar:default:
  image: ${DOCKER_REGISTRY}/env/sonarscanner:v1.02
  stage: sonar
  script:
    - /sonar-scanner/bin/sonar-scanner
        -Dsonar.projectKey=${PROJECT_GROUP}.${PROJECT_NAME}.${CI_COMMIT_REF_NAME}
        -Dsonar.sources=/builds/${PROJECT_GROUP}/${PROJECT_NAME}
        -Dsonar.projectName=${PROJECT_GROUP}.${PROJECT_NAME}.${CI_COMMIT_REF_NAME}
        -Dsonar.host.url=${SONAR_HOST}
        -Dsonar.login=f6008364db3b52711b641ab87d7981445f760545
        -Dsonar.gitlab.project_id=${CI_PROJECT_PATH}
        -Dsonar.gitlab.commit_sha=${CI_COMMIT_SHA}
        -Dsonar.gitlab.ref_name=${CI_COMMIT_REF_NAME}
  except:
    - test
    - tags
    - master
sonar:test:
  image: ${DOCKER_REGISTRY}/env/sonarscanner:v1.02
  stage: sonar
  script:
    - /sonar-scanner/bin/sonar-scanner
        -Dsonar.projectKey=${PROJECT_GROUP}.${PROJECT_NAME}.test
        -Dsonar.sources=/builds/${PROJECT_GROUP}/${PROJECT_NAME}
        -Dsonar.projectName=${PROJECT_GROUP}.${PROJECT_NAME}.test
        -Dsonar.host.url=${SONAR_HOST}
        -Dsonar.login=f6008364db3b52711b641ab87d7981445f760545
        -Dsonar.gitlab.project_id=${CI_PROJECT_PATH}
        -Dsonar.gitlab.commit_sha=${CI_COMMIT_SHA}
        -Dsonar.gitlab.ref_name=${CI_COMMIT_REF_NAME} | tee log.txt
    - grep -q 'desc=SonarQube reported QualityGate is ok' log.txt && exit 0 || exit -1
  only:
    - test
sonar:prod:
  image: ${DOCKER_REGISTRY}/env/sonarscanner:v1.02
  stage: sonar
  script:
    - /sonar-scanner/bin/sonar-scanner
        -Dsonar.projectKey=${PROJECT_GROUP}.${PROJECT_NAME}.prod
        -Dsonar.sources=/builds/${PROJECT_GROUP}/${PROJECT_NAME}
        -Dsonar.projectName=${PROJECT_GROUP}.${PROJECT_NAME}.prod
        -Dsonar.host.url=${SONAR_HOST}
        -Dsonar.login=f6008364db3b52711b641ab87d7981445f760545
        -Dsonar.gitlab.project_id=${CI_PROJECT_PATH}
        -Dsonar.gitlab.commit_sha=${CI_COMMIT_SHA}
        -Dsonar.gitlab.ref_name=${CI_COMMIT_REF_NAME} | tee log.txt
    - grep -q 'desc=SonarQube reported QualityGate is ok' log.txt && exit 0 || exit -1
  only:
    - tags


#测试环境
job_package:test:
  image: ${DOCKER_REGISTRY}/env/dind:v1.12
  stage: package
  script:
    - pwd
    - PROJECT_NAME=${CI_PROJECT_NAME}
    - wget ${STATIC_PATH}/test/${STATIC_PROJECT_NAME}/${STATIC_FILE_NAME}
    - if [ -d "${STATIC_CATALOG}" ]; then rm -fr ${STATIC_CATALOG}; fi
    - tar -zxvf ${STATIC_FILE_NAME}
    - rm -fr ${STATIC_FILE_NAME}
    - cp docker/* .
    - DOCKER_REPO=${DOCKER_REGISTRY}/${PROJECT_GROUP}/${PROJECT_NAME}:${CI_BUILD_REF_NAME}
    - docker build -t ${DOCKER_REPO} .
    - docker push ${DOCKER_REPO}
    - echo ${DOCKER_REPO}
  only:
    - test

job_deploy:test:
  image: ${DOCKER_REGISTRY}/env/rancher-compose:latest
  stage: deploy
  script:
    - PROJECT_NAME=${CI_PROJECT_NAME}
    - export PROJECT_VERSION=${CI_BUILD_REF_NAME}
    - cd docker
    - sed -i 's#${PROJECT_NAME}#'$PROJECT_NAME'#g' docker-compose.yml
    - sed -i 's#${PROJECT_ENV}#'$ENV_TEST'#g' docker-compose.yml
    - rancher-compose --debug --url http://172.17.17.44:8080/ --access-key F5CAF611778CDBFECC2C --secret-key ijVyHub3saBTvp9mReNP4fGXEcFGo7GCudoAAShd -p front-end-house up -d --force-upgrade --confirm-upgrade
  only:
    - test

# 正式环境
job_package:prod:
  image: ${DOCKER_REGISTRY}/env/dind:v1.12
  stage: package
  script:
    - pwd
    - PROJECT_NAME=${CI_PROJECT_NAME}
    - wget ${STATIC_PATH}/prod/${STATIC_PROJECT_NAME}/${STATIC_FILE_NAME}
    - if [ -d "${STATIC_CATALOG}" ]; then rm -fr ${STATIC_CATALOG}; fi
    - tar -zxvf ${STATIC_FILE_NAME}
    - rm -fr ${STATIC_FILE_NAME}
    - cp docker/* .
    - DOCKER_REPO=${DOCKER_REGISTRY}/${PROJECT_GROUP}/${PROJECT_NAME}:${CI_BUILD_REF_NAME}
    - docker build -t ${DOCKER_REPO} .
    - docker push ${DOCKER_REPO}
    - echo ${DOCKER_REPO}
  only:
    - tags

```