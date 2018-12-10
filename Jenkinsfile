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
    DOCKERHUB_CREDENTIAL_ID = 'dockerhub-id'
    GITHUB_CREDENTIAL_ID = 'github-id'
    KUBECONFIG_CREDENTIAL_ID = 'demo-kubeconfig'
    DOCKERHUB_NAMESPACE = 'kubesphere'
    GTIHUB_ACCOUNT = 'kubesphere'
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
          sh 'npm install -g cnpm --registry=https://registry.npm.taobao.org'
          sh 'cnpm i --no-package-lock'
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
    stage('build & push snapshot') {
      steps {
        container('nodejs') {
          sh 'yarn build'
          sh 'docker build -t docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER .'
          withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKERHUB_CREDENTIAL_ID" ,)]) {
            sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
            sh 'docker push  docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER '
          }
        }

      }
    }
    stage('push latest'){
       when{
         branch 'master'
       }
       steps{
         container('nodejs'){
           sh 'docker tag  docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
           sh 'docker push  docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
         }
       }
    }
    stage('deploy to dev') {
      when{
        branch 'master'
      }
      steps {
        input(id: 'deploy-to-dev', message: 'deploy to dev?')
        kubernetesDeploy(configs: 'deploy/dev/**', enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_CREDENTIAL_ID")
      }
    }
    stage('push with tag'){
      when{
        expression{
          return params.TAG_NAME =~ /v.*/
        }
      }
      steps {
         container('nodejs'){
         input(id: 'release-image-with-tag', message: 'release image with tag?')
           withCredentials([usernamePassword(credentialsId: "$GITHUB_CREDENTIAL_ID", passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
             sh 'git config --global user.email "kubesphere@yunify.com" '
             sh 'git config --global user.name "kubesphere" '
             sh 'git tag -a $TAG_NAME -m "$TAG_NAME" '
             sh 'git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/$GTIHUB_ACCOUNT/$APP_NAME.git --tags'
           }
         sh 'docker tag  docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME '
         sh 'docker push  docker.io/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME '
         }
      }
    }
    stage('deploy to production') {
      when{
        expression{
          return params.TAG_NAME =~ /v.*/
        }
      }
      steps {
        input(id: 'deploy-to-production', message: 'deploy to production?')
        kubernetesDeploy(configs: 'deploy/prod/**', enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_CREDENTIAL_ID")
      }
    }
  }

}
