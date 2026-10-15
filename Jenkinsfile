pipeline {
    agent any

    environment {
        IMAGE_NAME = 'sarvotham2006/hospital-appointment-portal'
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/sarvothaman16/hospital-appointment-portal.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Test Application') {
            steps {
                sh 'docker rm -f hospital-test || true'

                sh 'docker run -d --name hospital-test -p 8088:80 ${IMAGE_NAME}:${BUILD_NUMBER}'

                sh 'sleep 5'

                sh 'curl -f http://localhost:8088'

                sh 'docker rm -f hospital-test'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" |
                        docker login -u "$DOCKER_USER" --password-stdin

                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    try {
                        sh '''
                            kubectl set image deployment/hospital-deployment \
                            hospital-container=${IMAGE_NAME}:${BUILD_NUMBER}

                            kubectl rollout status \
                            deployment/hospital-deployment \
                            --timeout=120s
                        '''
                    } catch (Exception e) {
                        echo 'Deployment failed. Rolling back to the previous stable version.'

                        sh '''
                            kubectl rollout undo deployment/hospital-deployment

                            kubectl rollout status \
                            deployment/hospital-deployment \
                            --timeout=120s
                        '''

                        error('Deployment failed. Previous stable version restored.')
                    }
                }
            }
        }
    }
}
