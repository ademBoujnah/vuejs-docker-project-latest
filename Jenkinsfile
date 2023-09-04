pipeline {
    agent any

    environment {
        DOCKER_IMAGE_VERSION = 'V1'
        CURRENT_STAGE = ''
        // Define dockerImageTag at the top level
        DOCKER_IMAGE_TAG = ""
    }

    stages {
        stage('Build') {
            steps {
                script {
                    DOCKER_IMAGE_VERSION = "V${DOCKER_IMAGE_VERSION.replace('V', '').toInteger() + 1}"
                    // Assign the dockerImageTag at the top level
                    DOCKER_IMAGE_TAG = "ademboujnah/vuejs-app:${DOCKER_IMAGE_VERSION}"
                    
                    CURRENT_STAGE = 'Build'
                    try {
                        sh "docker build -t $DOCKER_IMAGE_TAG ."
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error("Build failed: ${e.message}")
                    }
                }
            }
        }

        stage('Code Analysis') {
            steps {
                script {
                    CURRENT_STAGE = 'Code Analysis'
                    def scannerHome = tool 'SonarQubeScanner'
                    try {
                        withSonarQubeEnv('SonarQube') {
                            sh "${scannerHome}/bin/sonar-scanner"
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error("Code Analysis failed: ${e.message}")
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    CURRENT_STAGE = 'Push to Docker Hub'
                    try {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                            sh "docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                            // Use the DOCKER_IMAGE_TAG variable defined at the top level
                            sh "docker push $DOCKER_IMAGE_TAG"
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error("Push to Docker Hub failed: ${e.message}")
                    }
                }
            }
        }
    }

    post {
        failure {
            script {
                def failedStageName = env.CURRENT_STAGE ?: "Unknown"

                emailext subject: "Pipeline Failed in Stage: ${currentBuild.fullDisplayName}",
                         body: "The pipeline '${currentBuild.fullDisplayName}' has failed in the '${failedStageName}' stage. Please investigate the issue.",
                         to: "adem.boujnah@esprit.tn",
                         mimeType: 'text/html'
            }
        }
    }
}
