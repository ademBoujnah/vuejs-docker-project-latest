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
                try {
                    // Code analysis steps here
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('SonarQube') {
                        // Run SonarQube code analysis
                        sh "${scannerHome}/bin/sonar-scanner"
                } catch (Exception e) {
                    STAGE_STATUS = 'FAILURE'
                    throw e
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
                try {
                    // Authenticate with Docker Hub and push the image
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                        sh "docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                        sh "docker push $DOCKER_IMAGE_TAG"}
                } catch (Exception e) {
                    STAGE_STATUS = 'FAILURE'
                    throw e
                }
            }
            post {
                always {
                    echo "Push to Docker Hub Stage Status: $STAGE_STATUS"
                }
            }
        }

        // Define other stages similarly...

    }

    post {
        failure {
            script {
                def failedStage = ''
                for (stage in currentBuild.rawBuild.getCauses()) {
                    if (stage.isInstanceOf(hudson.model.CauseOfInterruption.class)) {
                        failedStage = stage.getShortDescription()
                        break
                    }
                }

                if (failedStage == '') {
                    failedStage = 'Unknown Stage'
                }

                emailext subject: "Pipeline Failed in Stage: ${failedStage}",
                         body: "The pipeline '${currentBuild.fullDisplayName}' has failed in the '${failedStage}' stage. Please investigate the issue.",
                         to: "adem.boujnah@esprit.tn",
                         mimeType: 'text/html'
            }
        }
    }
}
