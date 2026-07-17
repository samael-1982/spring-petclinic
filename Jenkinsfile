  pipeline {
      agent {
          kubernetes {
              yaml '''
  apiVersion: v1
  kind: Pod
  spec:
    securityContext:
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
    containers:
      - name: maven
        image: eclipse-temurin:17-jdk
        command:
          - cat
        tty: true
      - name: kaniko
        image: gcr.io/kaniko-project/executor:v1.23.2-debug
        command:
          - cat
        tty: true
        volumeMounts:
          - name: kaniko-docker-config
            mountPath: /kaniko/.docker
    volumes:
      - name: kaniko-docker-config
        emptyDir: {}
  '''
          }
      }

      environment {
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
                  container('maven') {
                      sh './mvnw test'
                  }
              }
          }

          stage('Package') {
              steps {
                  container('maven') {
                      sh './mvnw package -DskipTests'
                  }
              }
          }

         stage('Build and Push Image') {
            steps {
                container('kaniko') {
                    withCredentials([usernamePassword(credentialsId: 'github-ghcr-token', usernameVariable: 'GHCR_USER', passwordVariable:
                    'GHCR_TOKEN')]) {
                        sh '''
                            set -e

                            echo "Preparing GHCR auth"
                            mkdir -p /kaniko/.docker

                            AUTH="$(printf '%s:%s' "$GHCR_USER" "$GHCR_TOKEN" | base64 | tr -d '\\n')"
                            printf '{"auths":{"ghcr.io":{"auth":"%s"}}}' "$AUTH" > /kaniko/.docker/config.json

                            echo "Building and pushing image: $IMAGE_NAME:$IMAGE_TAG"
                            /kaniko/executor \
                                --context "$WORKSPACE" \
                                --dockerfile "$WORKSPACE/Dockerfile" \
                                --destination "$IMAGE_NAME:$IMAGE_TAG" \
                                --verbosity=info

                            echo "Pushed image: $IMAGE_NAME:$IMAGE_TAG"
                        '''
                    }
                }
            }
         }
      }

      post {
          always {
            echo "Build is completed!"
            deleteDir()            
          }
      }
  }
