
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'abhijith006/online-bookstore'
        DEPLOYMENT = 'online-bookstore'
        CONTAINER_NAME = 'online-bookstore'
    }

    stages {
        stage('Clone Code') {
            steps {
                echo 'Code checked out from GitHub.'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .'
            }
        }

        stage('Test Application') {
            steps {
                bat 'docker run -d --name bookstore-test -p 8082:80 %DOCKER_IMAGE%:%BUILD_NUMBER%'
                bat 'powershell -NoProfile -Command "try { $r = Invoke-WebRequest http://localhost:8082 -UseBasicParsing; if ($r.StatusCode -ne 200) { exit 1 } } catch { exit 1 }"'
                bat 'docker rm -f bookstore-test'
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKER_TOKEN')]) {
                    bat '''
                        powershell -NoProfile -Command "$env:DOCKER_TOKEN | docker login -u 'abhijith006' --password-stdin"
                    '''
                    bat 'docker push %DOCKER_IMAGE%:%BUILD_NUMBER%'
                    bat 'docker logout'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    bat 'kubectl apply -f deployment.yaml'
                    bat 'kubectl set image deployment/%DEPLOYMENT% %CONTAINER_NAME%=%DOCKER_IMAGE%:%BUILD_NUMBER%'
                    bat 'kubectl rollout status deployment/%DEPLOYMENT% --timeout=120s'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    bat 'kubectl get deployment online-bookstore'
                    bat 'kubectl get pods -l app=online-bookstore'
                    bat 'kubectl get service online-bookstore-service'
                }
            }
        }
    }

    post {
        always {
            bat 'docker rm -f bookstore-test 2>nul || exit /b 0'
        }
    }
}
