pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    environment {
        DOCKER_IMAGE = "sagar520/devops-app"   
        GIT_REPO_NAME = "Spring-boot"
        GIT_USER_NAME = "MAJJISAGAR"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/MAJJISAGAR/Spring-boot.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Scanning project with SonarQube'
                sh 'ls -ltr'  
                sh '''
                    mvn sonar:sonar \
                    -Dsonar.host.url=http://18.212.157.206:9000 \
                    -Dsonar.login=squ_112bf6dfdebaf8d8534afda5c90adc4bbf3d2612
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} .'
            }
        }
        
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $DOCKER_IMAGE:${BUILD_NUMBER}'
            }
        }

        stage('Update Deployment File') {
            steps {
                echo 'Updating deployment.yaml with new image tag'
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "sagarmajji143@gmail.com"
                        git config user.name "MAJJISAGAR"

                        sed -i "s|image:.*|image: ${DOCKER_IMAGE}:${BUILD_NUMBER}|g" deployment.yaml

                        git add deployment.yaml
                        git commit -m "Update image to version ${BUILD_NUMBER}" || true

                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                    '''
                }
            }
        }
    }
}
