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
    K8S_NAMESPACE  = "devops"

    // ---- SonarQube ----
    SONARQUBE_SERVER = "sonarqube"       // Jenkins > Configure System > SonarQube servers (Name)
    SONAR_PROJECT_KEY = "devops-project" // must be unique in SonarQube
    SONAR_PROJECT_NAME = "Devops-Project"
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
        sh 'kubectl version --client'
      }
    }

    stage('Build JAR') {
      steps {
        sh 'mvn -B clean package -DskipTests'
      }
    }

   stage('SonarQube Analysis') {
  steps {
    withSonarQubeEnv("${SONARQUBE_SERVER}") {
      sh """
        mvn -B sonar:sonar \
          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
          -Dsonar.projectName='${SONAR_PROJECT_NAME}'
      """
    }
  }
}

stage('Quality Gate') {
  steps {
    timeout(time: 5, unit: 'MINUTES') {
      waitForQualityGate abortPipeline: true
    }
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
        sh 'docker compose down || true'
        sh 'docker compose up -d --build'
        sh 'docker compose ps'
      }
    }

    stage('Kubernetes Deploy') {
      steps {
        sh 'kubectl get nodes'
        sh """
          kubectl get ns ${K8S_NAMESPACE} >/dev/null 2>&1 || kubectl create namespace ${K8S_NAMESPACE}
        """
      }
    }

    stage('Deploy MySQL & Spring Boot on K8s') {
      steps {
        sh "kubectl apply -n ${K8S_NAMESPACE} -f k8s/mysql-deployment.yaml"
        sh "kubectl apply -n ${K8S_NAMESPACE} -f k8s/spring-deployment.yaml"
        sh "kubectl get pods -n ${K8S_NAMESPACE}"
        sh "kubectl get svc  -n ${K8S_NAMESPACE}"
      }
    }
  }

  post {
    success {
      emailext(
        to: 'aziz.gharbi@esprit.tn',
        subject: "✅ Jenkins SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Build succeeded ✅

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}
"""
      )
    }

    unstable {
      emailext(
        to: 'aziz.gharbi@esprit.tn',
        subject: "⚠️ Jenkins UNSTABLE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Build unstable ⚠️

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}
"""
      )
    }

    failure {
      emailext(
        to: 'aziz.gharbi@esprit.tn',
        subject: "❌ Jenkins FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Build failed ❌

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}

Check console output for details.
"""
      )
    }

    always {
      sh 'docker logout || true'
    }
  }
}
