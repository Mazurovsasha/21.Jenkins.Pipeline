pipeline {
    agent { label 'node01' }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Mazurovsasha/21.Jenkins.Pipeline.git'
            }
        }

        stage('Validate Dockerfile') {
            steps {
                sh 'docker run --rm -i hadolint/hadolint < app/Dockerfile'
            }
        }

        stage('Build and Test Image') {
            steps {
                sh 'docker build -t mazurovsasha/hw.21:v.1.0.0 ./app'
                sh 'docker run -d --name test-container3 -p 5000:5000 mazurovsasha/hw.21:v.1.0.0'
                sh 'sleep 15' 

                timeout(time: 5, unit: 'SECONDS') {
                    retry(3) {
                        script {
                            try {
                                sh 'docker exec test-container3 curl http://localhost:5000' 
                            } catch (Exception e) {
                                error "WebUI of application is not accessible"
                            }
                        }
                    }
                }

                sh 'docker stop $(docker ps -a -q -f ancestor=mazurovsasha/hw.21:v.1.0.0)'
                sh 'docker rm $(docker ps -a -q -f ancestor=mazurovsasha/hw.21:v.1.0.0)'
            }
        }

        stage('Push Image to Registry') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-credentials',
                            usernameVariable: 'REGISTRY_USERNAME',
                            passwordVariable: 'REGISTRY_PASSWORD'
                        )
                    ]) {
                        sh 'docker login -u $REGISTRY_USERNAME -p $REGISTRY_PASSWORD'
                        sh 'docker push mazurovsasha/hw.21:v.1.0.0'
                    }
                }
            }
        }

        stage('Deploy to Pre-Prod') {
            steps {
                input message: 'Approve Deployment to Pre-Prod', submitter: 'jenkins'

                // Deploy to Pre-Prod namespace
                // sh 'kubectl config use-context your-kubectl-context'
                sh 'kubectl apply -f app-pre-prod.yml --namespace pre-prod'

                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        // Test if deployment is successful
                        try {
                            sh 'kubectl rollout status deployment/myapp --namespace pre-prod'
                        } catch (Exception e) {
                            error "Deployment to Pre-Prod failed"
                        }

                        // Display message and prompt for approval
                        echo 'Deployment to Pre-Prod is successful'
                        input message: 'Approve Deployment to Prod', submitter: 'jenkins'
                    }
                }
            }
        }

        stage('Cleaning') {
            steps {
                // Remove deployment from Pre-Prod namespace
                sh 'kubectl delete deployment myapp --namespace pre-prod'
            }
        }
        stage('Deploy to Prod') {
            steps {
                // Deploy to Prod namespace
                sh 'kubectl apply -f app-prod.yml --namespace prod'

                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        // Test if deployment is successful
                        try {
                            sh 'kubectl rollout status deployment/myapp --namespace prod'
                        } catch (Exception e) {
                            error "Deployment to Prod failed"
                        }
                    }
                }
            }
        }

    }

    post {
            success {
                slackSend (channel: 'jenlins', color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        
            }
            failure {
                slackSend (channel: 'jenlins', color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
    }
}