pipeline{

    agent any

    environment {
        BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
    }

    stages{
        stage('Echo') {
            steps{
                sh "echo Je suis là ${car}"
                }
            }
        stage('Show Release') {
            steps {
                sh "cat /etc/os-release"
            }
        }
        stage('Initialisation') {
            steps {
                sh "make clean"
                sh "make venv" 
                sh "make install"
            }
        }

        stage('Unit Test') {
            steps {
                sh "make test"
            }
        }

        stage('Build docker image') {
            steps {
                sh "docker ps -a" 
                sh "echo $BRANCH_NAME"
                sh "echo $env.GIT_BRANCH"
                sh "printenv"
                sh "docker build -t hervlokossou/${BRANCH_NAME}_default_image ."
            }
        }


        stage('Test docker image') { 
            steps {
                sh "docker run -d -p 5000:8000 --name default_container hervlokossou/${BRANCH_NAME}_default_image"
            }
        }

                
        stage('Cleanup ') {
            steps {
                sh "docker container stop default_container"
                sh "docker container rm default_container" 
            }
        }

        stage("Push image to Docker Hub"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'DOCKER_CREDS', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]){
                    sh """
                    docker login  --username $USERNAME --password $PASSWORD && \
                    docker push hervlokossou/${BRANCH_NAME}_default_image:latest
                    """
                }
            }
        }
    } 

    post{
        always {
            script {
                    sh """
                        check_address='docker ps -a | default_container'
                        if [ -z "$check_address" ]
                        then
                            validuser=0
                        else
                            validuser=1
                            docker container stop default_container
                            docker container rm default_container
                        fi
                        echo $validuser
                        """
                }
            }
    }    
}
