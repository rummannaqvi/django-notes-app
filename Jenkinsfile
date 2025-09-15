pipeline {
    agent any

    environment {
        EC2_HOST = credentials('ec2-host')   // Secret Text credential with EC2 host/IP
    }

    stages {
        stage('Prepare SSH') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key',
                                                 keyFileVariable: 'EC2_KEYFILE',
                                                 usernameVariable: 'EC2_USER')]) {
                    sh '''
                        echo "✅ SSH key prepared for user: $EC2_USER"
                        echo "Connecting to host: $EC2_HOST"
                    '''
                }
            }
        }

        stage('Clone or Update Repo') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key',
                                                 keyFileVariable: 'EC2_KEYFILE',
                                                 usernameVariable: 'EC2_USER')]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $EC2_KEYFILE $EC2_USER@$EC2_HOST << 'EOF'
                            set -e
                            if [ -d "django-notes-app" ]; then
                                echo "📂 Repository exists, pulling latest changes..."
                                cd django-notes-app
                                git pull
                            else
                                echo "⬇️ Cloning repository..."
                                git clone https://github.com/rummannaqvi/django-notes-app.git
                                cd django-notes-app
                            fi
                        EOF
                    '''
                }
            }
        }

        stage('Stop Old Containers') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key',
                                                 keyFileVariable: 'EC2_KEYFILE',
                                                 usernameVariable: 'EC2_USER')]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $EC2_KEYFILE $EC2_USER@$EC2_HOST << 'EOF'
                            cd django-notes-app
                            echo "Stopping old containers..."
                            docker compose down || true
                        EOF
                    '''
                }
            }
        }

        stage('Build & Start Containers') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key',
                                                 keyFileVariable: 'EC2_KEYFILE',
                                                 usernameVariable: 'EC2_USER')]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $EC2_KEYFILE $EC2_USER@$EC2_HOST << 'EOF'
                            cd django-notes-app
                            echo "⚙️ Building images..."
                            docker compose build
                            echo "🚀 Starting containers..."
                            docker compose up -d
                        EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Deployment failed. Check logs."
        }
    }
}
