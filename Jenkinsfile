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
                sh 'docker build -t hw.21:v.0.3.0 ./app'
                sh 'docker run -d --name test-container3 -p 5000:5000 hw.21:v.0.3.0'
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

                sh 'docker stop $(docker ps -q -f ancestor=hw.21:v.0.3.0)'
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
                        sh 'docker push mazurovsasha/hw.21:v.0.3.0'
                    }
                }
            }
        }

        // stage('Deploy to Pre-Prod') {
        //     steps {
        //         input message: 'Approve Deployment to Pre-Prod', submitter: 'jenkins'

        //         // Deploy to Pre-Prod namespace
        //         sh 'kubectl config use-context your-kubectl-context'
        //         sh 'kubectl apply -f kubernetes-pre-prod.yml --namespace pre-prod'

        //         timeout(time: 5, unit: 'MINUTES') {
        //             script {
        //                 // Test if deployment is successful
        //                 try {
        //                     sh 'kubectl rollout status deployment/your-deployment --namespace pre-prod'
        //                 } catch (Exception e) {
        //                     error "Deployment to Pre-Prod failed"
        //                 }

        //                 // Display message and prompt for approval
        //                 echo 'Deployment to Pre-Prod is successful'
        //                 input message: 'Approve Deployment to Prod', submitter: 'jenkins'
        //             }
        //         }
        //     }
        // }

        // stage('Deploy to Prod') {
        //     steps {
        //         // Deploy to Prod namespace
        //         sh 'kubectl apply -f kubernetes-prod.yml --namespace prod'

        //         timeout(time: 5, unit: 'MINUTES') {
        //             script {
        //                 // Test if deployment is successful
        //                 try {
        //                     sh 'kubectl rollout status deployment/your-deployment --namespace prod'
        //                 } catch (Exception e) {
        //                     error "Deployment to Prod failed"
        //                 }
        //             }
        //         }
        //     }
        // }

        // stage('Cleaning') {
        //     steps {
        //         // Remove deployment from Pre-Prod namespace
        //         sh 'kubectl delete deployment your-deployment --namespace pre-prod'
        //     }
        // }

        // stage('Notification') {
        //     steps {
        //         // Send notification about successful deployment
        //         sh 'echo "Deployment to Prod is successful. Sending notification..."'
        //         // Add your notification command here
        //     }
        // }
    }
}