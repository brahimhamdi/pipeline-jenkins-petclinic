pipeline {
    agent any 
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        REMOTE_USER = 'tomcat'
        REMOTE_HOST = '192.168.56.103'
        REMOTE_PATH = '/tmp'
        TOMCAT_PATH = '/opt/tomcat/webapps'
    }
    
    
    stages{
        
        stage("Git Checkout"){
        steps{
            git branch: 'master', changelog: false, poll: false, url: 'https://github.com/brahimhamdi/pipeline-jenkins-petclinic'
        }
    }
    
    
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Test"){
            steps{
                sh "mvn test"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=Petclinic-SpringBoot \
                        -Dsonar.projectName='Petclinic-SpringBoot' \
                        -Dsonar.host.url=http://192.168.56.104:9000 \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }
        
       
         stage("Build"){
            steps{
                sh " mvn clean package"
                sh "sleep 10"
            }
        }
        
        
        
        stage("Deploy Artifacts to Nexus") {
           steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-credentials',
                    usernameVariable: 'NEXUS_USERNAME',
                    passwordVariable: 'NEXUS_PASSWORD'
                )]) {
                    sh """
                        mvn deploy:deploy-file \
                        -DgroupId=org.springframework.samples \
                        -DartifactId=spring-petclinic \
                        -Dversion=3.0.0 \
                        -Dpackaging=war \
                        -Dfile=target/spring-petclinic.war \
                        -DrepositoryId=nexus \
                        -Durl=http://192.168.56.102:8081/repository/maven-releases/ \
                        -DskipTests
                    """
                }
            }
        }
        
         stage('Copy War to Tomcat Server') {
            steps {
                sshagent(['tomcat-server']) {
                    sh 'scp -o StrictHostKeyChecking=no **/*.war ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_PATH}'
                }
            }
        }
        
        stage("Deploy WebApp") {
            steps {
                sshagent(['tomcat-server']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} '
                            sudo mv ${REMOTE_PATH}/*.war ${TOMCAT_PATH} && 
                            sudo chown -R tomcat:tomcat ${TOMCAT_PATH}
                        '
                    """
                }
            }
        }
   
    }
}

