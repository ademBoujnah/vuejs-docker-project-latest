pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_TAG = "ademboujnah/vuejs-app:latest"
        SONARQUBE_SCANNER_HOME = tool name: 'SonarQubeScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
    }
    
    stages {
        stage('Build') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh 'docker build -t $DOCKER_IMAGE_TAG .'
                }
            }
        }
        
        stage('Code Analysis') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script {
                        def scannerHome = tool 'SonarQubeScanner'
                        withSonarQubeEnv('SonarQube') {
                            sh "${scannerHome}/bin/sonar-scanner"
                        }
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                        sh "docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                        sh "docker push $DOCKER_IMAGE_TAG"
                    }
                }
            }
        }
    }
    
    post {
        failure {
            script {
                def failedStage = currentBuild.rawBuild.actions.find { it instanceof hudson.model.CauseAction }?.causes[0]?.shortDescription
                def errorMessage = "The pipeline '${currentBuild.fullDisplayName}' has failed at stage '${failedStage}'."
                
                emailext subject: "Pipeline Failed: ${currentBuild.fullDisplayName}",
                         body: "${errorMessage}\n\nLogs:\n${currentBuild.rawBuild.getLog(200)}",
                         to: "your@email.com",
                         mimeType: 'text/html'
            }
        }
    }
}
