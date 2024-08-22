pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag to deploy')
        string(name: 'IMAGE_NAME', defaultValue: 'kapsiki/kapsiki-erp', description: 'Docker image name to deploy')
        string(name: 'REMOTE_MACHINE_USER', defaultValue: 'root', description: 'Remote machine user')
        string(name: 'REMOTE_MACHINE_NAME', defaultValue: '212.132.115.76', description: 'Remote machine IP address')
        string(name: 'DOCKER_HUB_USERNAME', defaultValue: 'ghislain.kouete@kouete.com', description: 'Docker Hub username')
    }


    stages {
        stage('GIT CLONE') {
            steps {
                git branch: 'devops-58', url: 'https://github.com/kapsiki/kapsiki-axelor-open-suite-webapp-8.0.8.git'
            }
        }

        stage('GRADLE BUILD') {
            steps {
                sh './gradlew clean build -x test --warning-mode=none'
            }
        }

        stage('BUILD AND PUSH DOCKER IMAGE') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD', variable: 'DOCKER_HUB_PASSWORD')])  {
                        sh "echo $DOCKER_HUB_PASSWORD | docker login -u ${params.DOCKER_HUB_USERNAME} --password-stdin"
                        docker.build("${params.IMAGE_NAME}:${params.IMAGE_TAG}", "--pull --no-cache -f Dockerfile .")
                        sh "docker push ${params.IMAGE_NAME}:${params.IMAGE_TAG}"
                    }
                }
            }
        }

        stage('UPDATE AND DEPLOY TO DEMO-SERVER') {
            steps {
                script {
                    def buildTag = "${params.IMAGE_TAG}"
                    def remoteMachineUser = "${params.REMOTE_MACHINE_USER}"
                    def remoteMachineName = "${params.REMOTE_MACHINE_NAME}"
                    def TEMP_DIR = "working-dir"

                    sh """
                        ssh ${remoteMachineUser}@${remoteMachineName} "mkdir -p ~/${TEMP_DIR}"
                        scp docker-compose.yaml ${remoteMachineUser}@${remoteMachineName}:~/${TEMP_DIR}/

                        ssh  ${remoteMachineUser}@${remoteMachineName} << EOF
                        cd ~/${TEMP_DIR}
                        sed -i 's#image: .*#image: ${params.IMAGE_NAME}:${buildTag}#' docker-compose.yaml
                        echo -e "Deploying erp app publish script to ${params.REMOTE_MACHINE_NAME} remotely from jenkins build server"
                        docker-compose -f docker-compose.yaml down -v
                        docker-compose -f docker-compose.yaml up -d
                        rm -rf ~/${TEMP_DIR}
                    
                    """
                }
            }
        }
    }
}
