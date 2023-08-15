pipeline {
    agent any
    
    environment {
        NEXUS_REPO_URL = "127.0.0.1:8081/repository/vuejs-dockerized"  // Replace with your actual Nexus Repository URL
        NEXUS_REPO_NAME = "vuejs-dockerized"               // Replace with your actual Nexus Repository Name
        DOCKER_IMAGE_TAG = "latest"
        DOCKER_IMAGE_NAME = "vuejs-app"
    }
    
    stages {
        stage('Build') {
            steps {
                // Build the Vue.js app in a Docker container
                sh 'docker build -t $DOCKER_IMAGE_TAG .'
            }
        }
        
        stage('Push to Nexus') {
            steps {
                // Authenticate with your Nexus repository using credentials
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials-id1', passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USERNAME')]) {
                    //sh "docker login http:/127.0.0.1:8081 -u $NEXUS_USERNAME -p $NEXUS_PASSWORD"
                    sh "docker tag $DOCKER_IMAGE_NAME $NEXUS_REPO_URL:adem"
                    sh "docker push $NEXUS_REPO_URL/vuejs-app:adem"
                }
            }
        }
    }
}
