pipeline {
  agent any

  tools {
    jdk 'JAVA_HOME'
    maven 'M2_HOME'
  }

  environment {
    DOCKERHUB_USER = "azizgharbi205"
    APP_NAME       = "devops-project"
    IMAGE_TAG      = "1.0.0"
    IMAGE          = "${DOCKERHUB_USER}/${APP_NAME}:${IMAGE_TAG}"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main',
            url: 'https://github.com/G-Azz/Devops-Project.git'
      }
    }

    stage('Verify tools') {
      steps {
        sh 'java -version'
        sh 'mvn -version'
        sh 'docker --version'
        sh 'docker compose version'
      }
    }

    stage('Build JAR') {
      steps {
        sh 'mvn -B clean package -DskipTests'
      }
    }

    stage('DOCKER IMAGE') {
      steps {
        sh 'ls -la target || true'
        sh "docker build -t ${IMAGE} ."
      }
    }

    stage('DOCKER HUB') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DH_USER',
          passwordVariable: 'DH_TOKEN'
        )]) {
          sh """
            echo "\$DH_TOKEN" | docker login -u "\$DH_USER" --password-stdin
            docker push ${IMAGE}
            docker logout
          """
        }
      }
    }

    stage('DOCKER-COMPOSE') {
      steps {
        // expects docker-compose.yml at repo root
        sh 'docker compose down || true'
        sh 'docker compose up -d --build'
        sh 'docker compose ps'
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
    }
  }
}
