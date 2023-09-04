pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_TAG = "ademboujnah/vuejs-app:latest"
        SONARQUBE_SCANNER_HOME = tool name: 'SonarQubeScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'

    }
    
    stages {
        stage('Build') {
            steps {
                // Build the Vue.js app in a Docker container .
                sh 'docker build -t $DOCKER_IMAGE_TAG .'
            }
        }
         stage('Code Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScannerm'
                    withSonarQubeEnv('SonarQube') {
                        // Run SonarQube code analysis
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                // Authenticate with Docker Hub using credentials .
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                    sh "docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                    sh "docker push $DOCKER_IMAGE_TAG"
                }
            }
        }
        
  }
  post {
    failure {
        script {
            def failedStage = currentBuild.rawBuild.getCause(hudson.model.Failure.class)?.failedSteps[0]?.context?.get(hudson.model.Cause.UserIdCause.class)?.shortDescription ?: 'Unknown Stage'
            
            emailext subject: "Pipeline Failed in Stage: ${failedStage}",
                     body: "The pipeline '${currentBuild.fullDisplayName}' has failed in the '${failedStage}' stage. Please investigate the issue.",
                     to: "adem.boujnah@esprit.tn",
                     mimeType: 'text/html'
        }
    }
}
}
