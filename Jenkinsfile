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
                     sh "docker build -t levep79/petclinic-tomcat:${env.BUILD_NUMBER} ."
                 }
             }
       }
       stage('Run local Container') {
           agent any
           steps {
               sh 'docker rm -f petclinic-tomcat-temp || true'
               sh "docker run -d --network=bridge --name petclinic-tomcat-temp levep79/petclinic-tomcat:${env.BUILD_NUMBER}"
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
               sh "sleep 1h"
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
                        sh "docker push levep79/petclinic-tomcat:${env.BUILD_NUMBER}"
                    }
                }
           }
       }
   }
   post {
        failure {
            script{
                mail bcc: '', body: 'Build Failed', cc: '', from: 'jenkins@gmail.com', replyTo: '', subject: "${env.JOB_NAME} Failed (<${env.BUILD_URL}|Open>)", to: 'lev@gmail.com'
            }
        }
        success {
            script {
              mail bcc: '', body: 'Build Success', cc: '', from: 'jenkins@gmail.com', replyTo: '', subject: "New Image levep79/petclinic-tomcat:${env.BUILD_NUMBER} have been pushed Successfuly ", to: 'lev@gmail.com'
            }
        }
   }
}
