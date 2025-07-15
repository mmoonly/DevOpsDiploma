pipeline {
    agent any

    triggers {
        pollSCM('H/1 * * * *')
    }

    environment {
        DOCKER_IMAGE = "mmoonly/simple-java-maven-app:${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS_ID = 'docker-creds'
        DEPLOY_SERVER = "ubuntu@192.168.110.74"
        SSH_KEY = credentials('jenkins-to-app-ssh')
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                git branch: 'main', url: 'https://github.com/mmoonly/DevOpsDiploma.git'
                sh 'git submodule update --init --remote'
                sh 'ls -la'
                sh 'ls -la java-app'
            }
        }

        stage('Lint') {
            steps {
                dir('java-app') {
                    sh '/usr/share/maven/bin/mvn checkstyle:check'
                }
            }
        }

        stage('Build') {
            steps {
                dir('java-app') {
                    sh '/usr/share/maven/bin/mvn clean package'
                }
            }
        }

        stage('Test') {
            steps {
                dir('java-app') {
                    sh '/usr/share/maven/bin/mvn test'
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                dir('java-app') {
                    archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                dir('java-app') {
                    sh 'cp target/my-app-1.0-SNAPSHOT.jar . || cp target/*.jar .'
                    sh 'ls -la'
                    sh 'ls -l /var/run/docker.sock || echo "Docker socket not accessible"'

                    withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh '''
                            docker --version
                            docker build -t $DOCKER_IMAGE .
                            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                            docker push $DOCKER_IMAGE
                        '''
                    }
                }
            }
        }

        stage('Deploy to App Server') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY $DEPLOY_SERVER '
                        docker pull $DOCKER_IMAGE &&
                        docker stop app || true &&
                        docker rm app || true &&
                        docker run -d --name app -p 8080:8080 $DOCKER_IMAGE
                    '
                """
            }
        }

        stage('Notify') {
            steps {
                script {
                    def status = currentBuild.currentResult
                    def msg = "${status == 'SUCCESS' ? '✅' : '❌'} CI/CD завершён: билд #${BUILD_NUMBER}, образ: $DOCKER_IMAGE"
                    withCredentials([string(credentialsId: 'telegram-bot-token', variable: 'TOKEN')]) {
                        withCredentials([string(credentialsId: 'telegram-chat-id', variable: 'CHAT_ID')]) {
                            sh """
                                curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage \\
                                    -d chat_id=${CHAT_ID} -d text="${msg}" -d parse_mode="Markdown"
                            """
                        }
                    }
                }
            }
        }
    }
}
