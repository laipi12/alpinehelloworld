pipeline {
    agent none
    environment {
        IMAGE_NAME = "junior"
        IMAGE_TAG = "latest"
        DOCKERHUB_AUTH = credentials('dockerhub')
        ID_DOCKER = "${DOCKERHUB_AUTH_USR}"
        PORT_EXPOSED = "80"
    }
    stages {
        stage ('Build Image') {
            agent any
            steps {
                script {
                    sh 'docker build -t ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    echo "Clean Environment"
                    docker rm -f $IMAGE_NAME || echo "container does not exist"
                    docker run --name $IMAGE_NAME -d -p ${PORT_EXPOSED}:5000 -e PORT=5000 ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
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
                        curl http://172.17.0.1:${PORT_EXPOSED} | grep -q "Hello world!"
                    '''
                }
            }
        }

        stage('Clean Container') {
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

        stage ('Login and Push Image on docker hub') {
            agent any           
            steps {
                script {
                    sh '''
                        docker login -u $DOCKERHUB_AUTH_USR -p $DOCKERHUB_AUTH_PSW
                        docker push ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage ('Deploy in staging') {
            agent any
            environment {
                HOSTNAME_DEPLOY_STAGING = "ec2-13-218-164-164.compute-1.amazonaws.com"
            }
            steps {
                sshagent(credentials: ['SSH_AUTH_SERVER']) {
                    sh '''
                        [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                        ssh-keyscan -t rsa,dsa ${HOSTNAME_DEPLOY_STAGING} >> ~/.ssh/known_hosts
                        command1="docker login -u $DOCKERHUB_AUTH_USR -p $DOCKERHUB_AUTH_PSW"
                        command2="docker pull $DOCKERHUB_AUTH_USR/$IMAGE_NAME:$IMAGE_TAG"
                        command3="docker rm -f webapp || echo 'app does not exist'"
                        command4="docker run -d -p 80:5000 -e PORT=5000 --name webapp $DOCKERHUB_AUTH_USR/$IMAGE_NAME:$IMAGE_TAG"
                        ssh  ubuntu@${HOSTNAME_DEPLOY_STAGING} \
                            -o SendEnv=IMAGE_NAME \
                            -o SendEnv=IMAGE_TAG \
                            -o SendEnv=DOCKERHUB_AUTH_USR \
                            -o SendEnv=DOCKERHUB_AUTH_PSW \
                            -C "$command1 && $command2 && $command3  && $command4"
                    '''
                }
            }
        }

        stage ('Deploy in prod') {
            agent any
            environment {
                HOSTNAME_DEPLOY_PROD = "ec2-54-172-28-221.compute-1.amazonaws.com"
            }
            steps {
                sshagent(credentials: ['SSH_AUTH_SERVER']) {
                    sh '''
                        [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                        ssh-keyscan -t rsa,dsa ${HOSTNAME_DEPLOY_PROD} >> ~/.ssh/known_hosts
                        command1="docker login -u $DOCKERHUB_AUTH_USR -p $DOCKERHUB_AUTH_PSW"
                        command2="docker pull $DOCKERHUB_AUTH_USR/$IMAGE_NAME:$IMAGE_TAG"
                        command3="docker rm -f webapp || echo 'app does not exist'"
                        command4="docker run -d -p 80:5000 -e PORT=5000 --name webapp $DOCKERHUB_AUTH_USR/$IMAGE_NAME:$IMAGE_TAG"
                        ssh  ubuntu@${HOSTNAME_DEPLOY_PROD} \
                            -o SendEnv=IMAGE_NAME \
                            -o SendEnv=IMAGE_TAG \
                            -o SendEnv=DOCKERHUB_AUTH_USR \
                            -o SendEnv=DOCKERHUB_AUTH_PSW \
                            -C "$command1 && $command2 && $command3  && $command4"
                    '''
                }
            }
        }

    }
}
