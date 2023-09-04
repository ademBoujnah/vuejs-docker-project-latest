pipeline {
    agent any

    options {
        // Ensure that environment variables persist between builds
        buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '5'))
    }
    parameters {
        string(name: 'DOCKER_IMAGE_VERSION', defaultValue: '1', description: 'Docker image version')
    }

    environment {
        // Define dockerImageTag at the top level
        DOCKER_IMAGE_TAG = ""
        // Define CURRENT_STAGE at the top level
        CURRENT_STAGE = ''
        NAME = "ademboujnah/vuejs-app"
        VERSION = "1"        
    }

    stages {
        stage('Build') {
        steps {
             script {
                 
                 VERSION = params.DOCKER_IMAGE_VERSION.toInteger() + 1
                  try {
                      sh "docker build -t ${NAME} ."
                      sh "docker tag ${NAME}:latest ${NAME}:${VERSION}"
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
                            sh "docker push $NAME:$VERSION"
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
                def failedStageName = CURRENT_STAGE ?: "Unknown"

                emailext subject: "Pipeline Failed in Stage: ${currentBuild.fullDisplayName}",
                         body: "The pipeline '${currentBuild.fullDisplayName}' has failed in the '${failedStageName}' stage. Please investigate the issue.",
                         to: "adem.boujnah@esprit.tn",
                         mimeType: 'text/html'
            }
        }
    }
}
