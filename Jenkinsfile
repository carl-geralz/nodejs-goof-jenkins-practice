pipeline {
    agent none
    environment {
        DOCKERCRED = credentials('DockerLogin')
    }

    stages {
        stage('Build Node.js package') {
            agent {
                docker {
                    image 'node:slim'
                }
            }
            steps {
                sh 'npm install'
            }
        }

        stage('Build and Push Docker Image') {
            agent {
                docker {
                    image 'docker:dind'
                    args '--privileged --user root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'docker build -t carlgeralz/nodejs-goof-jenkins-practice:latest .'
                sh 'echo $DOCKERCRED_PSW | docker login -u $DOCKERCRED_USR --password-stdin'
                sh 'docker push carlgeralz/nodejs-goof-jenkins-practice:latest'
            }
        }

        stage('Deploy Docker Image') {
            agent {
                docker {
                    image 'kroniak/ssh-client'
                    args '--user root --privileged -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                script {
                    sh '''
                        ssh -i $SSH_KEY allmyfriends@192.168.145.50 '
                        echo $DOCKERCRED_PSW | docker login -u $DOCKERCRED_USR --password-stdin &&
                        docker pull carlgeralz/nodejs-goof-jenkins-practice:latest &&
                        docker rm --force nodejs-goof-jenkins-practice &&
                        docker run -it -d -p 3001:3001 --name nodejs-goof-jenkins-practice --network host carlgeralz/nodejs-goof-jenkins-practice:latest
                        '
                    '''
                }
            }
        }
    }
}
