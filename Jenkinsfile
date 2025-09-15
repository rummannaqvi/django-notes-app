pipeline {
    agent any

    environment {
        EC2_HOST = credentials('ec2-host')   // "Secret text" credential with your EC2 public IP or hostname
    }

    stages {
        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ['ec2-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@$EC2_HOST << 'EOF'
                            set -e
                            # If repo exists, update it; otherwise clone fresh
                            if [ -d "django-notes-app" ]; then
                                cd django-notes-app
                                git pull
                            else
                                git clone https://github.com/rummannaqvi/django-notes-app.git
                                cd django-notes-app
                            fi

                            # Stop old containers
                            docker compose down || true

                            # Build and start containers
                            docker compose build
                            docker compose up -d
                        EOF
                    '''
                }
            }
        }
    }
}
