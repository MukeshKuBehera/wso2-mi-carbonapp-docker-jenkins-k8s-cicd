pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('List services') {
            steps {
                sh 'ls -l services'
            }
        }

        stage('Build HealthcareService CAR') {
            steps {
                dir('services/HealthcareService') {
                    sh 'mvn clean package'
                }
            }
        }
    }
}

