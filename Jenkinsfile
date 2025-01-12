pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "skywalker-staging"
        PRODUCTION = "skywalker-production"
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t kevskywalker94/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Clean Environment"
                        docker rm -vf $IMAGE_NAME || echo "container doesn't exist"
                        docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 kevskywalker94/$IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                    '''
                }
            }
        }
        stage('Test image') {
            agent any
            steps {
                script {
                    sh '''
                       curl http://172.17.0.1:80 | grep -q "hello world!"
                    '''
                }
            }
        }
        stage('Clean container') {
            agent any
            steps {
                script {
                    sh '''
                       docker stop $IMAGE_NAME
                       docker rm $IMAGE_NAME
                    '''
                }
            }
        }
        stage('push image in staging and deploy it') {
            when {
                expression { GET_BRANCH == 'origin/master' }
            }
            agent {
                docker {
                    image 'franela/dind'
                    args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
                    }
            }

            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                       heroku container:login
                       heroku create $STAGING || echo "project already exist"
                       heroku container:push -a $STAGING web
                       heroku container:release -a $STAGING web

                    '''
                }
            }
        }
        stage('push image in production and deploy it') {
            when {
                expression { GET_BRANCH == 'origin/master' }
            }
            agent {
                docker {
                    image 'franela/dind'
                    args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
                    }
            }

            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                       heroku container:login
                       heroku create $PRODUCTION || echo "project already exist"
                       heroku container:push -a $PRODUCTION web
                       heroku container:release -a $PRODUCTION web

                    '''
                }
            }
        }
        
    }
  post {
       success {
         slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) - PROD URL => http://${PROD_APP_ENDPOINT} , STAGING URL => http://${STG_APP_ENDPOINT}")
         }
      failure {
         slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
          }   
    }
      
}

