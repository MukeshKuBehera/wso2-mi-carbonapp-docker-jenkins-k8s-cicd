pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('List Services') {
            steps {
                sh 'ls -l services'
            }
        }

        stage('Build All Services (CAR)') {
            steps {
                script {
                    // Find all service directories under services/
                    def services = sh(
                        script: "ls -d services/*/",
                        returnStdout: true
                    ).trim()

                    if (!services) {
                        error "No services found under services/ directory"
                    }

                    def svcList = services.split("\n")

                    for (svc in svcList) {
                        echo "Building service: ${svc}"
                        dir(svc) {
                            // Ensure mvnw is executable and build
                            sh 'chmod +x mvnw'
                            sh './mvnw clean package'
                        }
                    }
                }
            }
        }

        stage('Archive CAR Files') {
            steps {
                archiveArtifacts artifacts: 'services/**/target/*.car', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Build completed successfully. CAR files archived.'
        }
        failure {
            echo 'Build failed. Check logs above.'
        }
    }
}

