pipeline {
    agent any

    triggers {
        pollSCM('* * * * *')
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building application..."
                sh 'pip3 install --break-system-packages -r requirements.txt'
            }
        }

        stage('Test') {
            steps {
                echo "Running tests..."
                sh 'python3 -m unittest discover'
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
                sh 'docker build -t devsecops-app .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker rm -f devsecops-app || true'
                sh 'docker run -d -p 5001:5000 --name devsecops-app devsecops-app'
            }
        }
    }
}
