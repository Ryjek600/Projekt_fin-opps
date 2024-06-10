pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'hello-world-app'
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Ryjek600/Projekt_fin.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build(env.DOCKER_IMAGE)
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    docker.image("${env.DOCKER_IMAGE}:latest").run("-p 5000:5000")
                }
            }
        }
    }

    post {
        success {
            echo 'Build and Deployment Successful'
        }
        failure {
            script {
                echo 'Build or Deployment Failed - Rolling Back'
                rollback()
            }
        }
    }
}

def rollback() {
    script {
        // Define your rollback steps here
        // Example: Rollback to previous Docker image version
        try {
            docker.image("${env.DOCKER_IMAGE}:previous").run("-p 5000:5000")
            echo 'Rollback to previous version successful'
        } catch (Exception e) {
            echo 'Rollback failed'
        }

        // Example: Notify team
        emailext(
            to: 'r.piasecki.064@studms.ug.edu.pl',
            subject: "Jenkins Rollback Executed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Deployment failed and rollback has been executed. Please check the application status."
        )
    }
}

