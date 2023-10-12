pipeline {
    agent any

    environment {
        // Define dockerImageTag at the top level
        DOCKER_IMAGE_TAG = ""
        // Define CURRENT_STAGE at the top level
        CURRENT_STAGE = ''
        NAME = "ademboujnah/vuejs-app"
        VERSION = "${env.BUILD_ID}"    
    }

    stages {
        stage('Git Clone') {
        steps {
             script {
                 CURRENT_STAGE = 'Build'
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

        stage('SSH Into K8s Server') {
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

        stage('Put deployment files onto k8smaster') {
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
     stage('Deploy the application') {
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
     stage('Monitoring the application') {
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
