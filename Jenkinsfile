pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'OpenJDK'
        allure 'Allure'
    }

    environment {
        SONAR_PROJECT_KEY = 'myapp'
        SONAR_PROJECT_NAME = 'MY_Application'
        SONAR_HOST_URL = 'https://adc9-83-220-239-246.ngrok-free.app'
        SONAR_LOGIN = credentials('sonar-token')
    }

    stages {
        stage('Checkout') {
            steps {
                sh 'rm -rf myapp'
                sh 'git clone https://github.com/ArtemGonch/myapp.git'
            }
        }
        stage('Install libs') {
            steps {
                dir('myapp') {
                    sh 'apt-get update'
                    sh 'apt install -y maven'
                }
            }
        }

        stage('Build') {
            steps {
                dir('myapp') {
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'cd myapp'
                dir('myapp') {
                    sh 'mvn clean test'
                }
            }
            post {
                always {
                    allure([
                        includeProperties: false,
                        jdk: '',
                        properties: [],
                        reportBuildPolicy: 'ALWAYS',
                        results: [[path: 'myapp/allure-results']]
                    ])
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('myapp') {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh '''
                            mvn sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_LOGIN}
                        '''
                    }
                }
            }
        }
        stage('Install Docker') {
            steps {
                sh 'apt-get update'
                sh 'apt-get install -y docker.io'
                sh 'systemctl enable docker'
                sh 'dockerd > /dev/null 2>&1 &'

            }
        }
        stage('Docker Build') {
            steps {
                dir('myapp'){
                    script {
                        sh 'docker build -t artemgoncharov3012/myapp:latest .'
                    }
                }
            }
        }
        stage('Docker Push') {
            steps {
                dir('myapp') {
                    script {
                        withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                            sh 'echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin'
                        }
                        sh 'docker push artemgoncharov3012/myapp:latest'
                    }
                }
            }
        }
    }
}