pipeline {
  agent { label 'builder' }

  environment {
      DOCKER_HUB_CREDENTIALS = credentials('DockerPass')
      BACKEND_IMAGE = 'rushimagar1/backend'
      FRONTEND_IMAGE = 'rushimagar1/frontend'
      IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {
      stage('Checkout') {
          steps {
              git branch: 'main', url: 'https://github.com/rushimagar1/hrms-deployment.git'
          }
      }

      stage('Build Backend') {
          steps {
              dir('Backend_hrms') {
                  sh 'mvn clean package -DskipTests'
              }
          }
      }

      stage('Build Frontend') {
          steps {
              dir('frontend') {
                  sh 'npm install'
                  sh 'ng build --configuration production'
              }
          }
      }

      stage('Build & Push Docker Images') {
    parallel {
        stage('Backend Docker Image') {
            steps {
                dir('Backend_hrms') {
                    script {
                        def backendImage = docker.build("${BACKEND_IMAGE}:${IMAGE_TAG}")
                        docker.withRegistry('https://registry.hub.docker.com', 'DockerPass') {
                            backendImage.push()
                            backendImage.push('latest')
                        }
                    }
                }
            }
        }

        stage('Frontend Docker Image') {
            steps {
                dir('frontend') {
                    script {
                        def frontendImage = docker.build("${FRONTEND_IMAGE}:${IMAGE_TAG}")
                        docker.withRegistry('https://registry.hub.docker.com', 'DockerPass') {
                            frontendImage.push()
                            frontendImage.push('latest')
                        }
                    }
                }
            }
        }
    }
}


      stage('Deploy with Docker Compose') {
          steps {
              sh '''
                docker stop $(docker ps -q) || true
                docker-compose down || true
                docker-compose up -d
              '''
          }
      }
  }

  post {
      always {
          cleanWs()
      }
      success {
          echo 'HRMS deployment successful!'
      }
      failure {
          echo 'HRMS deployment failed!'
      }
  }
}
