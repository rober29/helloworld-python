pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        //git branch: 'main', url: 'https://github.com/rober29/helloworld-python.git'
      }
    }
    stage('SonarQube Analysis') {
      steps {
        sh './sonar-scanner -Dsonar.projectKey=helloworld-python -Dsonar.sources=. -Dsonar.host.url=http://localhost:9000 -Dsonar.login=7f551c9805e51c4c7e317186d60d1cab87bb97b1'
        }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "rober29/helloworld-python:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('dockerhub-pwd')
      }
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            sh 'docker push ${DOCKER_IMAGE}'
            }
        }
    }
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "helloworld-python"
            GIT_USER_NAME = "rober29"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "rober29@gmail.com"
                    git config user.name "rober29"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    ImageTag=helloworld-python:$(cat kubernetes/deployment.yml | grep image | awk -F ":" '{print $3}')
                    sed -i "s/${ImageTag}/helloworld-python:${BUILD_NUMBER}/g" kubernetes/deployment.yml
                    git add kubernetes/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
