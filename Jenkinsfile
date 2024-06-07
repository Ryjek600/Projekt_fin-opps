pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'hello-world-app'
        STAGING_CONTAINER_ID = ''
        PRODUCTION_CONTAINER_ID = ''
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Ryjek600/Projekt_fin.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                script {
                    docker.build(env.DOCKER_IMAGE)
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    docker.image("${env.DOCKER_IMAGE}:latest").inside {
                        sh 'python -m unittest discover'
                    }
                }
            }
        }

        stage('Package') {
            steps {
                script {
                    docker.image("${env.DOCKER_IMAGE}:latest").inside {
                        sh 'python setup.py sdist'
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    STAGING_CONTAINER_ID = docker.image("${env.DOCKER_IMAGE}:latest").run("-p 5000:5000 -e ENV=staging").id
                }
            }
        }

        stage('Integration Test') {
            steps {
                script {
                    try {
                        // Add integration test commands here
                        echo 'Running integration tests...'
                        // Example integration test command
                        sh 'curl -f http://localhost:5000'
                    } catch (Exception e) {
                        sh "docker stop ${STAGING_CONTAINER_ID}"
                        error 'Integration tests failed, rolling back deployment'
                    }
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                script {
                    try {
                        PRODUCTION_CONTAINER_ID = docker.image("${env.DOCKER_IMAGE}:latest").run("-p 5000:5000 -e ENV=production").id
                    } catch (Exception e) {
                        sh "docker stop ${PRODUCTION_CONTAINER_ID}"
                        error 'Deployment to production failed, rolling back'
                    }
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
                if (env.STAGING_CONTAINER_ID) {
                    sh "docker stop ${env.STAGING_CONTAINER_ID}"
                }
                if (env.PRODUCTION_CONTAINER_ID) {
                    sh "docker stop ${env.PRODUCTION_CONTAINER_ID}"
                }
                echo 'Build or Deployment Failed'
            }
        }
    }
}
