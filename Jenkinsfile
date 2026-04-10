pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    environment {
        DOCKER_IMAGE = "sagar123/devops-app"   
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
                withSonarQubeEnv('MySonarQube') {   
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.host.url=http://3.236.87.173:9000 \
                        -Dsonar.login=squ_fbc0c0d2d594e0658de1355b3741ee10bf3b51f1
                    '''
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} .'
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $DOCKER_IMAGE:${BUILD_NUMBER}'
            }
        }

        stage('Update Deployment File') {
            steps {
                echo 'Updating deployment.yml with new image tag'
                withCredentials([string(credentialsId: 'githubtoken', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "sagarmajji143@gmail.com"
                        git config user.name "MAJJISAGAR"

                        sed -i "s|image:.*|image: ${DOCKER_IMAGE}:${BUILD_NUMBER}|g" deploymentfiles/deployment.yml

                        git add deploymentfiles/deployment.yml
                        git commit -m "Update image to version ${BUILD_NUMBER}" || true

                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                    '''
                }
            }
        }
    }
}
