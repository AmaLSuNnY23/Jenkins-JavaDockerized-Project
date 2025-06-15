pipeline {
  agent any
  tools {
    maven 'maven'  // Use the exact name you gave in Jenkins Global Tool Configuration
  }
  environment {
    IMAGE = 'amalsunny27/jenkins-project'
    DOCKER_CREDS = credentials('Docker-cred')
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/AmaLSuNnY23/Jenkins-JavaDockerized-Project.git'
      }
    }
    stage('Build JAR') {
      steps {
        bat 'mvn clean package -DskipTests'
      }
    }
    stage('Generate Dockerfile') {
      steps {
        script {
          writeFile file: 'Dockerfile', text: """
FROM eclipse-temurin:17-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY \${JAR_FILE} app.jar
EXPOSE 7500
ENTRYPOINT ["java", "-jar", "/app.jar"]
"""
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          docker.build(IMAGE)
        }
      }
    }
    stage('Push to DockerHub') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'Docker-cred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            bat 'echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin'
            bat "docker push %IMAGE%:latest"
          }
        }
      }
    }
    stage('Run Container') {
      steps {
        bat 'docker rm -f javaapp || exit 0'
        bat "docker run -d -p 7500:8080 --name javaapp %IMAGE%:latest"
      }
    }
  }
  post {
    always {
      bat 'docker logout'
    }
  }
}
