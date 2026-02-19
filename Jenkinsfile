pipeline {
    agent any

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
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-17'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                script {
                    // Find all directories inside services/
                    def services = sh(
                        script: "ls -d services/*/",
                        returnStdout: true
                    ).trim().split("\n")

                    for (svc in services) {
                        echo "Building service: ${svc}"
                        dir(svc) {
                            sh 'mvn clean package'
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
}

