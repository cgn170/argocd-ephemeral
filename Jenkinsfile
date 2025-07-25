pipeline {

  agent {
    kubernetes {
      yamlFile 'kaniko-builder.yaml'
    }
  }

  options {
      buildDiscarder logRotator( 
                  daysToKeepStr: '16', 
                  numToKeepStr: '10'
          )
      disableConcurrentBuilds()
  }

  environment {
    APP_NAME = "argocd-ephemeral"
    DOCKER_REGISTRY = "harbor.devops.svc:8080/library"
    IMAGE_NAME = "${DOCKER_REGISTRY}" + "/" + "${APP_NAME}"
    DOCKERFILE_PATH = "Dockerfile"
    APP_NAME_LABEL = "${APP_NAME}"
    CONTAINER_NAME = "${APP_NAME}"
  }

  stages {

    stage("Checkout from SCM"){
      steps {
        script {
            sh """
              echo "Cloning repo with branch: ${GIT_BRANCH}"
            """
            checkout([
              $class: 'GitSCM',
              branches: scm.branches,
              extensions: scm.extensions + [[$class: 'CloneOption', noTags: false, reference: '', shallow: false]],
              userRemoteConfigs: scm.userRemoteConfigs
            ])
          }
        
      }
    }

    stage('Unit Testing') {
      steps {
        container(name: 'node', shell: '/bin/sh') {
          sh '''#!/bin/sh
            echo "Running Unit Tests"
          '''
        }
      }
    }

    stage('Code Analysis') {
        steps {
            sh """
            echo "Running Code Analysis"
            """
        }
    }

    stage("Get Version"){
      steps {
        script {
          env.HEAD_SHA = sh(returnStdout: true, script: 'git rev-parse --short=8 HEAD').trim()
          env.VERSION = env.HEAD_SHA
          echo "Branch: ${env.GIT_BRANCH}"
          echo "Git tag version: ${VERSION}"
          echo "Creating ${IMAGE_NAME}:${VERSION} container image"
        }
      }
    }

    stage('Build & Push with Kaniko') {
      steps {
            script {
                container(name: 'kaniko', shell: '/busybox/sh') {
                        sh '''#!/busybox/sh
                        /kaniko/executor --dockerfile `pwd`/${DOCKERFILE_PATH} \
                                        --context `pwd` \
                                        --destination=${IMAGE_NAME}:${VERSION} \
                                        --insecure \
                                        --skip-tls-verify
                        '''
                }
            }
        }
    }


  }
  
}
