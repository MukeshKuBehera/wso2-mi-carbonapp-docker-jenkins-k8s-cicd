🚀 Automated WSO2 MI CI/CD Pipeline (Pull → Build → Deploy)
============================================================
<img width="1536" height="1024" alt="CI CD_pipeline_for_WSO2_MI_CAR_deployment_using_Jenkins_Docker" src="https://github.com/user-attachments/assets/cf64f9f7-329d-4911-8286-58b1a455cfaf" />

📌 Overview
------------
This project demonstrates a real-world CI/CD workflow for WSO2 Micro Integrator where:

✔ Developer pulls code from Git
✔ Makes changes in VS Code
✔ Pushes back to GitHub
✔ Jenkins automatically builds .car
✔ Installs WSO2 MI (if not running)
✔ Deploys .car to CarbonApps directory

🏗️ Architecture
-----------------
Developer (VS Code)
        │
   git pull / push
        │
        ▼
   GitHub Repository
        │
        ▼
   Jenkins Pipeline
        │
        ├── Checkout Code
        ├── Build CAR (mvnw)
        ├── Check / Start WSO2 MI Docker
        ├── Deploy CAR
        └── Verify Deployment
        │
        ▼
WSO2 Micro Integrator (Docker)

⚙️ Prerequisites
------------------
Java 11+
Docker
Git
Jenkins (running)
VS Code + WSO2 MI Extension

🔄 COMPLETE REAL WORKFLOW (STEP BY STEP)
------------------------------------------
🔹 Step 1 — Clone Project (First Time)

cd ~/wso2-workspace/mi

git clone https://github.com/<your-username>/wso2-mi-carbonapp-docker-jenkins-k8s-cicd.git

cd wso2-mi-carbonapp-docker-jenkins-k8s-cicd

🔹 Step 2 — Pull Latest Code (IMPORTANT 🔥)

👉 Always pull before making changes

git pull origin main

🔹 Step 3 — Open in VS Code & Modify

code .

Make changes:

API XML

Endpoints

Sequences

YAML definitions

Example path:

services/HealthcareService/src/main/wso2mi/artifacts/apis/

🔹 Step 4 — Build Locally (Optional Testing)

cd services/HealthcareService

chmod +x mvnw
./mvnw clean package

Check CAR:

ls target/

🔹 Step 5 — Commit Changes

cd ~/wso2-workspace/mi/wso2-mi-carbonapp-docker-jenkins-k8s-cicd

git add .

git commit -m "Updated Healthcare API flow"

🔹 Step 6 — Push to GitHub

👉 If using SSH (Recommended)

git push origin main

👉 If using HTTPS (with token)

git push https://<username>:<token>@github.com/<username>/repo.git

🔹 Step 7 — Jenkins Pipeline Trigger

👉 Jenkins automatically starts after push

OR manually:

Build Now (Jenkins UI)
-----------------------
⚙️ Jenkins Pipeline Logic

🔹 Step 8 — Checkout Code

checkout scm

🔹 Step 9 — Build CAR Files

./mvnw clean package

Generated:

services/*/target/*.car

🔹 Step 10 — Check WSO2 MI (Auto Install if Not Running)

docker ps | grep wso2mi

If NOT running:

docker run -d \
--name wso2mi \
-p 8290:8290 \
-p 8253:8253 \
wso2/wso2mi:4.4.0

🔹 Step 11 — Deploy CAR File to MI

docker cp services/*/target/*.car \

wso2mi:/home/wso2carbon/wso2mi-4.4.0/repository/deployment/server/carbonapps/

🔹 Step 12 — Verify Deployment

docker logs -f wso2mi

Expected:

Deploying Carbon Application : HealthcareService_1.0.0.car

🔥 FINAL Jenkinsfile (FULL AUTOMATION)
------------------------------------------
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

        stage('Ensure WSO2 MI Running') {
            steps {
                sh '''
                if ! docker ps | grep -q wso2mi; then
                    echo "WSO2 MI container not running. Starting..."
                    
                    docker run -d \
                    --name wso2mi \
                    -p 8290:8290 \
                    -p 8253:8253 \
                    wso2/wso2mi:4.4.0

                    echo "Waiting for MI to start..."
                    sleep 20
                else
                    echo "WSO2 MI already running"
                fi
                '''
            }
        }

        stage('Deploy CAR Files to MI') {
            steps {
                sh '''
                echo "Deploying CAR files..."

                for file in services/*/target/*.car; do
                    echo "Copying $file to WSO2 MI..."
                    
                    docker cp $file \
                    wso2mi:/home/wso2carbon/wso2mi-4.4.0/repository/deployment/server/carbonapps/
                done
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "Checking deployment logs..."
                docker logs wso2mi | tail -n 50
                '''
            }
        }
    }

    post {
        success {
            echo 'Build & Deployment completed successfully 🚀'
        }
        failure {
            echo 'Pipeline failed ❌ Check logs above.'
        }
    }
}
🌐 API Testing
---------------
curl http://localhost:8290/healthcare/querydoctor/cardiology

## 🔧 Jenkins Pipeline Success
<img width="1279" height="591" alt="Jenkins_CI CD_pipeline" src="https://github.com/user-attachments/assets/ff329d8d-c5e6-4ecb-a4c5-1571794147d7" />

## 📦 Generated CAR File
<img width="1034" height="104" alt="CAR File Generated" src="https://github.com/user-attachments/assets/dcfbb537-24dd-410b-a02f-411967c54801" />

##  Docker Container Running
<img width="1137" height="161" alt="Docker Container Running" src="https://github.com/user-attachments/assets/f1b3b07e-297b-4b96-b728-d996e1dc9263" />

##  Deployemt Logs
<img width="1034" height="635" alt="Docker_WSO2MI_LOGS" src="https://github.com/user-attachments/assets/a2afdd15-2489-4886-9968-f1002f2a6cbf" />


🎯 Key Highlights
-------------------
Pull → Modify → Push → Auto Deploy

No manual CAR deployment

Jenkins handles everything

Auto-starts WSO2 MI if not running

Supports multiple services

🚀 Benefits
-------------
✔ Fully automated pipeline
✔ Faster development cycle
✔ Zero manual deployment
✔ Scalable architecture
✔ Production-ready approach

🔮 Future Scope
----------------
Kubernetes Deployment (K8s)

API Testing Automation (Postman/Newman)

Kafka Integration

Redis Caching

Monitoring (Prometheus + Grafana)

👨‍💻 Author
-----------
Mukesh Behera
Senior Software Engineer
