pipeline {
  agent {
    node {
      label 'nodejs'
    }
  }
  parameters{
     string(name:'TAG_NAME',defaultValue: '',description:'')
  }
  environment {
    ORG = 'runzexia'
    APP_NAME = 'devops-docs-sample'
  }
  stages {
    stage('checkout scm') {
      steps {
        checkout(scm)
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
    stage('build & push snapshot image ') {
      when{
        branch 'master'
      }
      steps {
        container('nodejs') {
          sh 'yarn build'
          sh 'docker build -t docker.io/$ORG/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER .'
          withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : 'dockerhub' ,)]) {
            sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
            sh 'docker push  docker.io/$ORG/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER '
          }
        }

      }
    }
    stage('push latest image'){
       when{
         branch 'master'
       }
       steps{
         container('nodejs'){
           sh 'docker tag  docker.io/$ORG/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER docker.io/$ORG/$APP_NAME:latest '
           sh 'docker push  docker.io/$ORG/$APP_NAME:latest '
         }
       }
    }
    stage('deploy to dev?') {
      when{
        branch 'master'
      }
      steps {
        input(id: 'deploy-to-dev', message: 'deploy to dev?')
      }
    }
    stage('deploy to dev') {
      when{
        branch 'master'
      }
      steps {
        kubernetesDeploy(configs: 'deploy/dev/**', enableConfigSubstitution: true, kubeconfigId: 'kubeconfig')
      }
    }
    stage('release image with tag?'){
        when{
            tag 'v*'
        }
      steps {
        input(id: 'release-image-with-tag', message: 'release image with tag?')
      }
    }
    stage('push image with tag'){
        when{
            tag 'v*'
        }
        steps {
           container('nodejs'){
           sh 'docker tag  docker.io/$ORG/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER docker.io/$ORG/$APP_NAME:$TAG_NAME '
           sh 'docker push  docker.io/$ORG/$APP_NAME:$TAG_NAME '
           }
        }
    }
    stage('deploy to production?') {
      when{
        tag 'v*'
      }
      steps {
        input(id: 'deploy-to-production', message: 'deploy to production?')
      }
    }
    stage('deploy to production') {
      when{
        tag 'v*'
      }
      steps {
        kubernetesDeploy(configs: 'deploy/production/**', enableConfigSubstitution: true, kubeconfigId: 'kubeconfig')
      }
    }
  }

}
