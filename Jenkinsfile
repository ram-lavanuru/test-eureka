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
            SONAR_TOKEN = credentials('sonar-creds')
            SONAR_URL = "http://34.57.70.242:9000"
    }
    stages {
        stage('build') {
            steps {
                echo "*****builiding the ${env.APPLICATION_NAME} application"
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('sonar scan') {
            steps {
                echo "**performing sonar scan***"
                withSonarQubeEnv('SonarQube') {  //SonarQube is same as the name system under manage jenkins
                sh """
                mvn sonar:sonar \
                -Dsonar.project=i27-eureka \
                -Dsonar.host.url=${env.SONAR_URL} \
                -Dsonar.login=${SONAR_TOKEN}
                """
                } 
                timeout (time:2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
