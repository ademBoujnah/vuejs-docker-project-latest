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
                        sh 'docker buildh -t $DOCKER_IMAGE_TAG .'
                    } catch (Exception e) {
                        currentBuild.result = 'Build'
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
                        currentBuild.result = 'Code Analysis'
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
                        currentBuild.result = 'Push to Docker Hub'
                        error("Push to Docker Hub failed: ${e.message}")
                    }
                }
            }
        }
    }

    post {
    failure {
        script {
            def failedStageName = null

            // Check which stage failed by examining the currentBuild.result
            if (currentBuild.resultIsWorseThan('SUCCESS')) {
                def failedStage = currentBuild.rawBuild.getCause(hudson.model.Cause$UpstreamCause)
                if (failedStage) {
                    failedStageName = failedStage.shortDescription
                }
            }

            if (failedStageName == null) {
                failedStageName = "Unknown"
            }

            emailext subject: "Pipeline Failed in Stage: ${currentBuild.fullDisplayName}",
                     body: "The pipeline '${currentBuild.fullDisplayName}' has failed in the '${failedStageName}' stage. Please investigate the issue.",
                     to: "adem.boujnah@esprit.tn",
                     mimeType: 'text/html'
        }
    }
 }

}
