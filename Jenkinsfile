  pipeline {
      agent any

      environment {
          REGISTRY = 'ghcr.io'
          IMAGE_NAME = 'ghcr.io/samael-1982/spring-petclinic'
          IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
      }

      stages {
          stage('Checkout') {
              steps {
                  checkout scm
              }
          }

          stage('Test') {
              steps {
                  sh './mvnw test'
              }
          }

          stage('Build Image') {
              steps {
                  sh './mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=${IMAGE_NAME}:${IMAGE_TAG}'
              }
          }

          stage('Push Image') {
              steps {
                  withCredentials([usernamePassword(credentialsId: 'github-ghcr-token', usernameVariable: 'GHCR_USER',
                  passwordVariable: 'GHCR_TOKEN')]) {
                      sh '''
                          echo "$GHCR_TOKEN" | docker login ghcr.io -u "$GHCR_USER" --password-stdin
                          docker push "$IMAGE_NAME:$IMAGE_TAG"
                          docker logout ghcr.io
                      '''
                  }
              }
          }
      }

      post {
          always {
              cleanWs()
          }
      }
  }

