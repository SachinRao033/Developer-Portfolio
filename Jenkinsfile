pipeline {
    agent any

    environment {
        PROJECT_DIR = "/home/ubuntu/Developer-Portfolio"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${BRANCH}",
                    credentialsId: 'github-creds',
                    url: "${REPO_URL}"
            }
        }

        stage('Prepare Project') {
            steps {
                sh """
                sudo rm -rf ${PROJECT_DIR}
                sudo mkdir -p /home/ubuntu
                sudo cp -r \$WORKSPACE ${PROJECT_DIR}
                sudo chown -R ubuntu:ubuntu ${PROJECT_DIR}
                """
            }
        }

        stage('Create Environment Files') {
            steps {
                sh """
                cd ${PROJECT_DIR}

                cat > .env <<EOF
MYSQL_ROOT_PASSWORD=root123
EOF

                cat > App_Server/.env <<EOF
MYSQL_URL=mysql+pymysql://root:root123@mysql:3306/portfolio_db
SESSION_SECRET=portfolio-admin-session-secret-2024
ADMIN_PASSWORD=YOUR_SECURE_ADMIN_PASSWORD
ADMIN_EMAIL=admin@example.com
PORT=8000
EOF

                cat > Portfolio/.env <<EOF
VITE_EMAILJS_SERVICE_ID=
VITE_EMAILJS_TEMPLATE_ID=
VITE_EMAILJS_PUBLIC_KEY=
VITE_API_URL=
ADMIN_PASSWORD=admin123
JWT_SECRET=change-this-secret-in-production
PORT=3000
EOF
                """
            }
        }

        stage('Cleanup Docker') {
            steps {
                sh """
                cd ${PROJECT_DIR}

                docker compose down || true
                docker system prune -a --volumes --force
                """
            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                cd ${PROJECT_DIR}
                docker compose build --no-cache
                """
            }
        }

        stage('Deploy Containers') {
            steps {
                sh """
                cd ${PROJECT_DIR}
                docker compose up -d
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                sleep 20
                docker ps
                """
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Portfolio deployed successfully!'
        }

        failure {
            echo 'FAILED: Deployment failed.'
        }
    }
}
