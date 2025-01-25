pipeline {
    agent {
        label 'k8s-slave'
    }
    tools {
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }
    environment {
            APPLICATION_NAME = "eureka"
    }
    stages {
        stage('build') {
            steps {
                echo "*****builiding the ${env.APPLICATION_NAME} application"
                sh 'mvn clean package -DskipTests=true'
            }
        }
    }
}