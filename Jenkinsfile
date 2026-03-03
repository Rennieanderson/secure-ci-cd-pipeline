pipeline {
    agent any

    triggers {
        pollSCM('* * * * *')
    }

    environment {
        IMAGE_NAME = "devsecops-app"
        CONTAINER_NAME = "devsecops-app"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test (Python Container)') {
            agent {
                docker {
                    image 'python:3.9'
                    args '-u root'
                }
            }
            steps {
                echo "Installing dependencies..."
                sh 'pip install --no-cache-dir -r requirements.txt'

                echo "Running unit tests..."
                sh 'python -m unittest discover'
            }
        }

        stage('Manual Approval (IAM)') {
            steps {

                emailext(
                    subject: "🚨 Approval Required for Deployment",
                    body: """
Hello Admin,

Deployment is waiting for approval.

Click the link below to approve:
${env.BUILD_URL}

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}

Thanks,
Jenkins CI/CD
""",
                    recipientProviders: [requestor()]
                )

                input message: "Approve deployment to production?", ok: "Deploy"
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker rm -f $CONTAINER_NAME || true
                docker run -d -p 5001:5000 --name $CONTAINER_NAME $IMAGE_NAME
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline executed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
