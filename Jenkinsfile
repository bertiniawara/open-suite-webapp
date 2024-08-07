pipeline {
    agent any

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/bertiniawara/open-suite-webapp.git'
            }
        }

        stage('Change Directory') {
            steps {
                dir('open-suite-webapp') {
                    sh 'cd open-suite-webapp'
                    sh 'cd modules/axelor-open-suite'
                    sh 'git submodule init'
                    sh 'git submodule update --depth=1'
                    sh 'cd ../..'
                    sh 'git submodule update'
                }
            }
        }

        stage('Build') {
            steps {
                dir('open-suite-webapp') {
                    sh './gradlew clean build -x test'
                }
            }
        }

        stage('Docker Compose') {
            steps {
                dir('open-suite-webapp') {
                    sh 'docker-compose down -v'
                    sh 'docker-compose up --build'
                }
            }
        }
    }
}
