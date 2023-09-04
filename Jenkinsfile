pipeline {
    agent any

    environment {
         DOCKER_IMAGE_VERSION = 'V1'
        // Initialize a variable to track the current stage
        CURRENT_STAGE = ''
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Increment the version counter
                    DOCKER_IMAGE_VERSION = "V${DOCKER_IMAGE_VERSION.replace('V', '').toInteger() + 1}"
                    def dockerImageTag = "ademboujnah/vuejs-app:${DOCKER_IMAGE_VERSION}"

                    // Set the current stage name
                    CURRENT_STAGE = 'Build'
                    try {
                        // Build the Vue.js app in a Docker container and tag it with the version
                        sh "docker build -t $dockerImageTag ."
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
                    // Set the current stage name
                    CURRENT_STAGE = 'Code Analysis'
                    def scannerHome = tool 'SonarQubeScanner'
                    try {
                        withSonarQubeEnv('SonarQube') {
                            // Run SonarQube code analysis
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
                    // Set the current stage name
                    CURRENT_STAGE = 'Push to Docker Hub'
                    try {
                        // Authenticate with Docker Hub using credentials.
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                            sh "docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                            sh "docker push $dockerImageTag"
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
                // Use the custom CURRENT_STAGE variable to capture the failed stage name
                def failedStageName = env.CURRENT_STAGE ?: "Unknown"

                emailext subject: "Pipeline Failed in Stage: ${currentBuild.fullDisplayName}",
                         body: "The pipeline '${currentBuild.fullDisplayName}' has failed in the '${failedStageName}' stage. Please investigate the issue.",
                         to: "adem.boujnah@esprit.tn",
                         mimeType: 'text/html'
            }
        }
    }
}
