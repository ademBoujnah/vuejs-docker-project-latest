pipeline {
    agent any

    environment {
        DOCKER_IMAGE_TAG = "ademboujnah/vuejs-app:latest"
    }

    stages {
        stage('Build') {
            steps {
                script {
                    try {
                        // Build the Vue.js app in a Docker container.
                        sh 'docker build -t $DOCKER_IMAGE_TAG .'
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
                    try {
                        // Authenticate with Docker Hub using credentials.
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                            sh "docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
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
                emailext subject: "Pipeline Failed: ${currentBuild.fullDisplayName}",
                         body: "The pipeline '${currentBuild.fullDisplayName}' has failed. Please investigate the issue.",
                         to: "adem.boujnah@esprit.tn",
                         mimeType: 'text/html'
            }
        }
    }
}
