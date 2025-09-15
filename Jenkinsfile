pipeline {
    agent any

    environment {
        EC2_HOST = credentials('ec2-host')   // secret text with your EC2 host/IP
    }

    stages {
        stage('Deploy to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key',
                                                 keyFileVariable: 'EC2_KEYFILE',
                                                 usernameVariable: 'EC2_USER')]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $EC2_KEYFILE $EC2_USER@$EC2_HOST << 'EOF'
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
}
