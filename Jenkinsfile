  pipeline {
    agent {
      kubernetes {
        yaml '''
  apiVersion: v1
  kind: Pod
  spec:
    securityContext:
      fsGroup: 1000
    containers:
      - name: maven
        image: eclipse-temurin:17-jdk
        command:
          - cat
        tty: true
        securityContext:
          runAsUser: 1000
          runAsGroup: 1000
      - name: kaniko
        image: gcr.io/kaniko-project/executor:v1.23.2-debug
        command:
          - cat
        tty: true
        securityContext:
          runAsUser: 0
          runAsGroup: 0
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
    REGISTRY    = 'ghcr.io'
    IMAGE_NAME  = 'ghcr.io/samael-1982/spring-petclinic'     
    GIT_BRANCH  = 'main'
    CHART_DIR   = 'charts/petclinic'
    VALUES_FILE = 'charts/petclinic/values.yaml'
    // PATH = "${env.WORKSPACE}/bin:${env.PATH}"
  }

  stages {
    stage('Checkout') {
        steps {
            checkout scm
        }
    }
    stage('Prepare') {
      steps {
        script {
            env.IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
            echo "Image tag: ${env.IMAGE_TAG}"
        }
      }
    }
    stage('Install yq') {
      steps {
        container('maven') {
          sh '''
            set -e
            echo "Purpose of this stage is to install yq tool into workspace bin directory if it is not already installed."
            mkdir -p "$WORKSPACE/bin"

            if [ ! -f "$WORKSPACE/bin/yq" ]; then
                echo "Installing yq..."
                wget -q \
                  https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 \
                  -O "$WORKSPACE/bin/yq"
                chmod +x "$WORKSPACE/bin/yq"
            fi

            "$WORKSPACE/bin/yq" --version
          '''
        }
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
    stage('Update Helm values') {
      steps {
        container('maven') {
          sh '''
            set -e

            echo "Updating image tag to $IMAGE_TAG"
            echo "Workspace: $WORKSPACE"
            ls -l "$WORKSPACE/bin"

            "$WORKSPACE/bin/yq" -i '
              .image.tag = strenv(IMAGE_TAG)
            ' "$VALUES_FILE"

            echo "Updated values.yaml:"
            cat "$VALUES_FILE"
          '''
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