pipeline {
    agent any

    environment {
        DOCKER_IMAGE_TAG = "ademboujnah/vuejs-app:latest"
    }

    stages {
        stage('Build') {
            environment {
                STAGE_STATUS = 'SUCCESS'
            }
            steps {
                try {
                    // Build the Vue.js app in a Docker container.
                    sh 'docker build -t $DOCKER_IMAGE_TAG .'
                } catch (Exception e) {
                    STAGE_STATUS = 'FAILURE'
                    throw e
                }
            }
            post {
                always {
                    echo "Build Stage Status: $STAGE_STATUS"
                }
            }
        }

        stage('Code Analysis') {
            environment {
                STAGE_STATUS = 'SUCCESS'
            }
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('SonarQube') {
                        // Run SonarQube code analysis
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
            post {
                always {
                    echo "Code Analysis Stage Status: $STAGE_STATUS"
                }
            }
        }

        stage('Push to Docker Hub') {
            environment {
                STAGE_STATUS = 'SUCCESS'
            }
            steps {
                // Authenticate with Docker Hub using credentials.
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                    try {
                        sh "docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                        sh "docker push $DOCKER_IMAGE_TAG"
                    } catch (Exception e) {
                        STAGE_STATUS = 'FAILURE'
                        throw e
                    }
                }
            }
            post {
                always {
                    echo "Push to Docker Hub Stage Status: $STAGE_STATUS"
                }
            }
        }
    }

    post {
        failure {
            script {
                def failedStages = []
                if (STAGE_STATUS == 'FAILURE') {
                    failedStages.add("Build")
                }
                if (STAGE_STATUS == 'FAILURE') {
                    failedStages.add("Code Analysis")
                }
                if (STAGE_STATUS == 'FAILURE') {
                    failedStages.add("Push to Docker Hub")
                }

                def failedStageStr = failedStages.join(", ")
                if (failedStageStr.isEmpty()) {
                    failedStageStr = 'Unknown Stage'
                }

                emailext subject: "Pipeline Failed in Stage(s): ${failedStageStr}",
                    body: "The pipeline '${currentBuild.fullDisplayName}' has failed in the stage(s): ${failedStageStr}. Please investigate the issue.",
                    to: "adem.boujnah@esprit.tn",
                    mimeType: 'text/html'
            }
        }
    }
}
