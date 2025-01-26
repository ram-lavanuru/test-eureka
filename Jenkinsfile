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
        stage('sonar scan') {
            steps {
                echo "**performing sonar scan***"
                sh """
                mvn sonar:sonar \
                -Dsonar.project=i27-eureka \
                -Dsonar.host.url=http://34.57.70.242:9000 \
                -Dsonar.login=squ_dd6a4e93a918b4097cc3feb21d0507fc570c6728
                """
            }
        }
    }
}
