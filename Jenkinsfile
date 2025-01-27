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
            SONAR_URL = "http://35.193.175.156:9000"
            //https://www.jenkins.io/doc/pipeline/steps/pipeline-utility-steps/#readmavenpom-read-a-maven-project-file
            //if any issue with readMaevnPom, make sure install pipeline utility steps plugin
            POM_VERSION = readMavenPom().getVersion()
            POM_PACKAGING = readMavenPom().getPackaging()
            DOCKER_HUB = "docker.io/venkat315"
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
        stage('Docker build') {
            steps {
                //existing artifcat format: i27-eureka-0.0.1-SNAPSHOT.jar
                //my destination artifact format: i27-eureka-buildnumber-branchname.jar
                echo "My Jar source: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                echo "My Jar destination: i27-${env.APPLICATION_NAME}-${BUILD_NUMBER}-${BRANCH_NAME}.${env.POM_PACKAGING}"
                sh """
                echo "****building docker image*****"
                pwd
                ls -la
                docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${BUILD_NUMBER}-${BRANCH_NAME}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${APPLICATION_NAME}:${GIT_COMMIT}
                # docker.io/venkat315/eureka:
                """
            }
        }
    }
}
