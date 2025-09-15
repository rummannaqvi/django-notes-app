pipeline {
    agent any

    environment {
        EC2_HOST     = credentials('ec2-host')      // Jenkins credential (secret text)
        EC2_SSH_KEY  = credentials('ec2-ssh-key')   // Jenkins credential (SSH private key)
    }

    stages {
        stage('Checkout Repo') {
            steps {
                checkout scm
            }
        }

        stage('Setup SSH') {
            steps {
                sh '''
                    mkdir -p ~/.ssh
                    echo "$EC2_SSH_KEY" > ~/.ssh/ec2_key.pem
                    chmod 600 ~/.ssh/ec2_key.pem
                    ssh-keyscan -H $EC2_HOST >> ~/.ssh/known_hosts
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                    ssh -i ~/.ssh/ec2_key.pem ubuntu@$EC2_HOST << 'EOF'
                        set -e
                        if [ -d "django-notes-app" ]; then
                            cd django-notes-app
                            git pull
                        else
                            git clone https://github.com/rummannaqvi/django-notes-app.git
                            cd django-notes-app
                        fi
                        docker compose down || true
                        docker compose build
                        docker compose up -d
                    EOF
                '''
            }
        }
    }
}
