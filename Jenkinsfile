pipeline {
  agent {
    node {
      label 'nodejs'
    }
  environment {
    ORG = 'runzexia'
    APP_NAME = 'devops-docs-sample'
  }
  }
  stages {
    stage('checkout scm') {
      steps {
        git(url: 'https://github.com/kubesphere/docs.kubesphere.io', changelog: true, poll: false)
      }
    }
    stage('get dependencies') {
      steps {
        container('nodejs') {
          sh 'yarn'
        }

      }
    }
    stage('unit test') {
      steps {
        container('nodejs') {
          sh 'yarn test'
        }

      }
    }
    stage('build') {
      steps {
        container('nodejs') {
          sh 'yarn build'
        }

      }
    }
    stage('build & push snapshot docker image ') {
      steps {
        container('nodejs') {
          sh 'docker build -t docker.io/$ORG/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER .'
        withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : 'dockerhub' ,)]) {
          sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'ß
          sh 'docker push  docker.io/$ORG/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER '
        }
       }
      }
    }
    stage('deploy to dev?') {
      steps {
        input(id: 'deploy-to-dev', message: 'deploy to dev?')
      }
    }
    stage('deploy to dev') {
      steps {
        kubernetesDeploy(configs: 'deploy/dev/**', enableConfigSubstitution: true, kubeconfigId: 'kubeconfig')
      }
    }
  }

}
