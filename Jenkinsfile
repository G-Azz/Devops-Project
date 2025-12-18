pipeline {
  agent any

  tools {
    jdk 'JAVA_HOME'
    maven 'M2_HOME'
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
      }
    }

    stage('Compile') {
      steps {
        sh 'mvn -B clean compile'
      }
    }
  }
}
