pipeline {
  agent any

  environment {
    DOCKER_IMAGE_NAME = "betiniawara842/kapsiki-axelor"
    DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  stages {
    stage('Git Clone') {
      steps {
        git branch: 'main', url: 'https://github.com/bertiniawara/open-suite-webapp.git'
      }
    }

    stage('Build') {
      steps {
        sh './gradlew clean build -x test'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          def dockerImage = docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}", "--no-cache -f Dockerfile .")
          withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD', variable: 'DOCKER_HUB_PASSWORD')]) {
            sh 'docker login -u betiniawara@gmail.com -p $DOCKER_HUB_PASSWORD'
            dockerImage.push() 
            }
          }
      }
    }
     stage('Update Docker Compose') {
        steps{
            script {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    withCredentials([usernamePassword(credentialsId: 'GIT_HUB_CREDENTIALS')]) {
                        sh "git config user.email betiniawara@gmail.com"
                        sh "git config user.name bertiniawara"
            
                        sh "cat docker-compose.yaml"
                        sh "sed -i 's+${DOCKER_IMAGE_NAME}:.*+${DOCKER_IMAGE_NAME}:${DOCKERTAG}+g' docker-compose.yaml"
                        sh "cat docker-compose.yaml"
                        sh "git add ."
                        sh "git commit -m 'Update Docker Compose file with new image tag:${DOCKER_IMAGE_TAG}'"
                        sh "git push origin HEAD:main"
                    }
                }
              }
            }
          }
    stage('Deploy') {
      steps {
        script {
          sh "docker-compose down -v"
          sh "docker-compose up -d"
        }
      }
    }
  }
}
