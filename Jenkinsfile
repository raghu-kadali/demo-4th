pipeline {
    agent none

    environment {
        IMAGE_NAME = "myapp"
    }

    stages {

        stage('Clone & Build Java Artifact on Master') {
            agent { label 'built-in' }     // <-- MASTER NODE LABEL
            steps {
                echo "Cloning repo & building Java app on Master node..."

                // Clone your GitHub project
                git 'https://github.com/raghu-kadali/demo-4th.git'

                // Build JAR file
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    echo "Stashing artifact for Slave node..."
                    stash includes: 'target/*jar-with-dependencies.jar', name: 'appJar'
                }
            }
        }

        stage('Build & Push Docker Image on Slave') {
            agent { label 'node1' }     // <-- YOUR SLAVE NODE LABEL
            steps {
                echo "Unstashing JAR on Slave node..."
                unstash 'appJar'

                withCredentials([usernamePassword(
                    credentialsId: 'docker_creds',   // <-- DOCKER HUB CREDENTIAL ID
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    echo "Logging into Docker Hub..."
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'

                    echo "Building Docker image..."
                    sh 'docker build -t $DOCKER_USER/$IMAGE_NAME:latest .'

                    echo "Pushing Docker image to Docker Hub..."
                    sh 'docker push $DOCKER_USER/$IMAGE_NAME:latest'
                }
            }
        }
    }
}
