pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_TAG = "ademboujnah/vuejs-app:latest"
        SONARQUBE_SCANNER_HOME = tool name: 'SonarQubeScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
    }
    
    stages {
        stage('Build') {
            steps {
                script {
                    try {
                        sh 'docker build -t $DOCKER_IMAGE_TAG .'
                    } catch (Exception buildException) {
                        currentBuild.result = 'FAILURE'
                        error "Build stage failed: ${buildException}"
                    }
                }
            }
        }
        
        stage('Code Analysis') {
            steps {
                script {
                    try {
                        def scannerHome = tool 'SonarQubeScannera'
                        withSonarQubeEnv('SonarQube') {
                            sh "${scannerHome}/bin/sonar-scanner"
                        }
                    } catch (Exception analysisException) {
                        currentBuild.result = 'FAILURE'
                        error "Code Analysis stage failed: ${analysisException}"
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    try {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                            sh "docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                            sh "docker push $DOCKER_IMAGE_TAG"
                        }
                    } catch (Exception pushException) {
                        currentBuild.result = 'FAILURE'
                        error "Push to Docker Hub stage failed: ${pushException}"
                    }
                }
            }
        }
    }
    
    post {
        failure {
            script {
                def failedStage = null
                def errorMessage = null
                
                if (currentBuild.resultIsBetterOrEqualTo('FAILURE')) {
                    if (currentBuild.rawBuild.causes.find { it instanceof hudson.model.Cause.UserIdCause }) {
                        errorMessage = "This pipeline was manually triggered."
                    } else {
                        errorMessage = "The pipeline has failed."
                    }
                    
                    if (currentBuild.rawBuild.actions.find { it instanceof hudson.model.CauseAction }) {
                        failedStage = currentBuild.rawBuild.actions.find { it instanceof hudson.model.CauseAction }.causes[0].shortDescription
                    }
                }
                
                emailext subject: "Pipeline Failed: ${currentBuild.fullDisplayName}",
                         body: "The pipeline '${currentBuild.fullDisplayName}' has failed at stage '${failedStage}'.\n\n${errorMessage}\n\nLogs:\n${currentBuild.rawBuild.getLog(200)}",
                         to: "adem.boujnah@esprit.tn",
                         mimeType: 'text/html'
            }
        }
    }
}
