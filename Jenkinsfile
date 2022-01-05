pipeline {
    agent none
    stages {
       stage('Build') {
           agent {
               docker {
                   image 'maven:3.5.0'
               }
           }
           steps {
                   sh 'mvn clean install -DskipTests=true -B'
           }
       }
       stage('Build container') {
             agent any
             steps {
                 script {
                     sh "docker build -t levep79/petclinic-tomcat:${env.BRANCH_NAME}-${env.BUILD_NUMBER} ."
                 }
             }
       }
       stage('Run local Container') {
           agent any
           steps {
               sh 'docker rm -f petclinic-tomcat-temp || true'
               sh "docker run -d --network=bridge --name petclinic-tomcat-temp levep79/petclinic-tomcat:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
           }
       }
       stage('Smoke-Test') {
           agent {
               docker {
                   image 'maven:3.5.0'
                   args '--network=bridge'
               }
           }
           steps {
               sh "cd regression-suite"
               sh "mvn clean -B test -DPETCLINIC_URL=http://petclinic-tomcat:8080/petclinic/"
           }
       }
       stage('Stop local container') {
           agent any
           steps {
               sh 'docker rm -f petclinic-tomcat-temp || true'
           }
       }
       stage('Push to dockerhub') {
           agent any
           steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-lev', url:''){
                       sh "docker push levep79/petclinic-tomcat:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    }
                }
           }
       }
   }
   post {
        failure {
            script{
                mail bcc: '', body: 'Build Failed', cc: '', from: 'jenkins@my_jenkins', replyTo: '', subject: '"${env.JOB_NAME} Failed (<${env.BUILD_URL}|Open>)"', to: 'lev@gmail.com'
            }
        }
        success {
            mail bcc: '', body: 'Build Success', cc: '', from: 'jenkins@my_jenkins', replyTo: '', subject: '"${env.JOB_NAME} Success (<${env.BUILD_URL}|Open>)"', to: 'lev@gmail.com'
        }
   }
}
