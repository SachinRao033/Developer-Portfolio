pipeline {
    agent any

    environment {
        PROJECT_DIR = "/home/ubuntu/Developer-Portfolio"
        REPO_URL    = "https://github.com/SachinRao033/Developer-Portfolio.git"
        BRANCH      = "main"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH}",
                    credentialsId: 'github-creds',
                    url: "${env.REPO_URL}"
            }
        }

        stage('Prepare Project') {
            steps {
                sh '''
                sudo rm -rf $PROJECT_DIR
                sudo mkdir -p /home/ubuntu
                sudo cp -r "$WORKSPACE" "$PROJECT_DIR"
                sudo chown -R ubuntu:ubuntu "$PROJECT_DIR"
                '''
            }
        }

        stage('Create Environment Files') {
            steps {
                sh '''
                cd "$PROJECT_DIR"

                # Root .env
                cat > .env <<EOF
MYSQL_ROOT_PASSWORD=root123
EOF

                # Backend .env
                mkdir -p App_Server
                cat > App_Server/.env <<EOF
MYSQL_URL=mysql+pymysql://root:root123@mysql:3306/portfolio_db
SESSION_SECRET=portfolio-admin-session-secret-2024
ADMIN_PASSWORD=admin123
ADMIN_EMAIL=admin@example.com
PORT=8000
ENVIRONMENT=development
CORS_ORIGINS=http://15.207.25.177:3000,http://localhost:3000
EOF

                # Frontend .env
                mkdir -p Portfolio
                cat > Portfolio/.env <<EOF
VITE_API_URL=http://15.207.25.177:8000
PORT=3000
EOF
                '''
            }
        }

        stage('Cleanup Docker') {
            steps {
                sh '''
                cd "$PROJECT_DIR"

                docker compose down || true
                docker system prune -a --volumes --force
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                cd "$PROJECT_DIR"
                docker compose build --no-cache
                '''
            }
        }

        stage('Deploy Containers') {
            steps {
                sh '''
                cd "$PROJECT_DIR"
                docker compose up -d
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                cd "$PROJECT_DIR"

                echo "Waiting for containers..."
                sleep 20

                docker ps
                docker compose ps
                docker compose logs backend --tail=30
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Developer Portfolio deployed successfully!'
        }

        failure {
            echo 'FAILED: Deployment failed.'
        }

        always {
            sh 'docker image prune -f || true'
        }
    }
}
