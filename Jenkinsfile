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
                sh 'ls -l services || true'
            }
        }

        stage('Build All Services (CAR)') {
            steps {
                script {
                    // List only real service directories (exclude Jenkins @tmp folders)
                    def services = sh(
                        script: "ls -d services/*/ 2>/dev/null | grep -v '@tmp' || true",
                        returnStdout: true
                    ).trim()

                    if (!services) {
                        error "No services found under services/ directory"
                    }

                    def svcList = services.split("\n")

                    for (svc in svcList) {
                        echo "Building service: ${svc}"
                        dir(svc) {
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

