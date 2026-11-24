pipeline {
    agent any

    parameters {
        string(name: 'BACKEND_TAG',  defaultValue: 'latest', description: 'Backend image tag')
        string(name: 'FRONTEND_TAG', defaultValue: 'latest', description: 'Frontend image tag')
    }

    environment {
        DEPLOY_HOST = 'ubuntu@13.204.65.190'
        APP_DIR     = '/home/ubuntu/todoapp'
    }

    stages {
        stage('Copy Files to Server') {
            steps {
                withCredentials([file(credentialsId: 'backend-env', variable: 'ENVFILE')]) {
                    sshagent(['prod-server-key']) {
                        sh '''
                          ssh -o StrictHostKeyChecking=no $DEPLOY_HOST "mkdir -p $APP_DIR && rm -f $APP_DIR/backend.env"
                          scp -o StrictHostKeyChecking=no compose.yml $DEPLOY_HOST:$APP_DIR/docker-compose.yml
                          scp -o StrictHostKeyChecking=no $ENVFILE $DEPLOY_HOST:$APP_DIR/backend.env
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['prod-server-key']) {
                    sh '''
                      ssh -o StrictHostKeyChecking=no $DEPLOY_HOST "
                        cd $APP_DIR &&
                        export BACKEND_TAG=$BACKEND_TAG FRONTEND_TAG=$FRONTEND_TAG &&
                        docker compose pull &&
                        docker compose up -d
                      "
                    '''
                }
            }
        }
    }
}