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
            // SONAR_URL = "http://35.223.190.169:9000"
            //https://www.jenkins.io/doc/pipeline/steps/pipeline-utility-steps/#readmavenpom-read-a-maven-project-file
            //if any issue with readMaevnPom, make sure install pipeline utility steps plugin
            POM_VERSION = readMavenPom().getVersion()
            POM_PACKAGING = readMavenPom().getPackaging()
            DOCKER_HUB = "docker.io/venkat315"
            DOCKER_CREDS = credentials('docker-creds')
            
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
                sh 'env | grep SONAR'
                sh """
                mvn sonar:sonar \
                -Dsonar.project=i27-eureka \
                #-Dsonar.host.url=${env.SONAR_URL} \
                #-Dsonar.login=${SONAR_TOKEN}
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
                cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
                docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${APPLICATION_NAME}:${GIT_COMMIT} ./.cicd
                # docker.io/venkat315/eureka:
                echo "********login to doker registry****"
                docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}
                docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}
                """
            }
        }
        stage('Deploy to dev') {
            steps {
                script {
                    dockerDeploy('dev', '5761', '8761').call()
                }
                    }
                    
                }
        stage('Deploy to test') {
            steps {
                script {
                    dockerDeploy('tst', '6761', '8761').call()
                }
                    }
                    
                }
        stage('Deploy to stage') {
            steps {
                script {
                    dockerDeploy('stg', '7761', '8761').call()
                }
                    }
                    
                }

        stage('Deploy to prod') {
            steps {
                script {
                    dockerDeploy(prd, 8761, 8761).call()
                }
                    }
                    
                }
                
            }
        }

              
  def dockerDeploy(envDeploy, hostPort, containerPort) {
    return {
      echo "**deploying to $envDeploy server****"
        withCredentials([usernamePassword(credentialsId: 'ram-docker-vm-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            script {
            sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$dev_ip \"docker pull ${env.DOCKER_HUB}/${APPLICATION_NAME}:${GIT_COMMIT}\""
            try {
                sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$dev_ip docker stop ${env.APPLICATION_NAME}-$envDeploy"
                sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$dev_ip docker rm ${env.APPLICATION_NAME}-$envDeploy"
            }
            catch(err) {
                echo "Error Caught: $err"
            }

            //deploy to dev

            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$dev_ip docker run -dit --name ${env.APPLICATION_NAME}-$envDeploy -p $hostPort:$containerPort ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
            
            }


        } 
    }
  }
            
        
