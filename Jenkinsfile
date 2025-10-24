pipeline {
  agent {
    label 'jenkins-agent'
  }

  tools {
    nodejs "NodeJS"
    dockerTool "Docker"
  }

  environment {
    DOCKER_CREDENTIALS = credentials("dockerhub")
    IMAGE_NAME = "kevin1208/jobber-review"
    IMAGE_TAG = "stable-${BUILD_NUMBER}"
  }

  stages {
    stage("Cleanup Workspace") {
      steps {
        cleanWs()
      }
    }

    stage("Checkout") {
      steps {
        git branch: 'master', credentialsId: 'github', url: 'https://github.com/kevindeveloper95/jobapp-review'
      }
    }

    stage("Install Dependencies") {
      steps {
        script {
          // Check if npm is available, if not use Docker
          def npmCheck = sh(script: 'which npm || echo "npm not found"', returnStdout: true).trim()
          if (npmCheck == "npm not found") {
            echo "Node.js not found, using Docker..."
            sh '''
              # Use Docker to run npm commands
              docker run --rm -v $(pwd):/app -w /app node:18-alpine npm install
            '''
          } else {
            sh 'npm install'
          }
        }
      }
    }

    stage("Lint Check") {
      steps {
        script {
          def npmCheck = sh(script: 'which npm || echo "npm not found"', returnStdout: true).trim()
          if (npmCheck == "npm not found") {
            sh 'docker run --rm -v $(pwd):/app -w /app node:18-alpine npm run lint:check'
          } else {
            sh 'npm run lint:check'
          }
        }
      }
    }

    stage("Code Format Check") {
      steps {
        script {
          def npmCheck = sh(script: 'which npm || echo "npm not found"', returnStdout: true).trim()
          if (npmCheck == "npm not found") {
            sh 'docker run --rm -v $(pwd):/app -w /app node:18-alpine npm run prettier:check'
          } else {
            sh 'npm run prettier:check'
          }
        }
      }
    }

    stage("Unit Test") {
      steps {
        script {
          def npmCheck = sh(script: 'which npm || echo "npm not found"', returnStdout: true).trim()
          if (npmCheck == "npm not found") {
            sh 'docker run --rm -v $(pwd):/app -w /app node:18-alpine npm run test'
          } else {
            sh 'npm run test'
          }
        }
      }
    }

    stage("Build and Push") {
      steps {
        sh 'docker login -u $DOCKER_CREDENTIALS_USR --password $DOCKER_CREDENTIALS_PSW'
        sh "docker build -t $IMAGE_NAME ."
        sh "docker tag $IMAGE_NAME $IMAGE_NAME:$IMAGE_TAG"
        sh "docker tag $IMAGE_NAME $IMAGE_NAME:stable"
        sh "docker push $IMAGE_NAME:$IMAGE_TAG"
        sh "docker push $IMAGE_NAME:stable"
      }
    }

    stage("Clean Artifacts") {
      steps {
        sh "docker rmi $IMAGE_NAME:$IMAGE_TAG"
        sh "docker rmi $IMAGE_NAME:stable"
      }
    }
  }

  post {
    success {
      echo "✅ Build succeeded! Build #${env.BUILD_NUMBER}"
      echo "🎉 Jobber Review service deployed successfully!"
      echo "📦 Docker image: ${env.IMAGE_NAME}:${env.IMAGE_TAG}"
    }
    failure {
      echo "❌ Build failed! Build #${env.BUILD_NUMBER}"
      echo "🔧 Check the logs for more details"
      echo "📋 Common issues:"
      echo "   - Check if all dependencies are installed"
      echo "   - Verify Docker credentials"
      echo "   - Check if the repository is accessible"
    }
  }
 }