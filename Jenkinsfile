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
                     sh "docker build -t levep79/petclinic-tomcat:${env.BRANCH_NAME} ."
                     if ( env.BRANCH_NAME == 'master' ) {
                         containerVersion = getVersionFromContainer("levep79/petclinic-tomcat:${env.BRANCH_NAME}")
                         failIfVersionExists("levep79","petclinic-tomcat",containerVersion)
                         sh "docker build -t levep79/petclinic-tomcat:${containerVersion} ."
                     }
                 }
             }
       }
       stage('Run local Container') {
           agent any
           steps {
               sh 'docker rm -f petclinic-tomcat-temp || true'
               sh "docker run -d --network=bridge --name petclinic-tomcat-temp levep79/petclinic-tomcat:${env.BRANCH_NAME}"
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
                        if ( env.BRANCH_NAME == 'master' )
                            sh "docker push levep79/petclinic-tomcat:${containerVersion}"
                        else sh "docker push levep79/petclinic-tomcat:${env.BRANCH_NAME}"
                    }
                }
           }
       }
   }
   post {
        failure {
            script{
                mail bcc: '', body: '', cc: '', from: '', replyTo: '', subject: '"${env.JOB_NAME} Failed (<${env.BUILD_URL}|Open>)"', to: 'lev@gmail.com'
            }
        }
        success {
            mail bcc: '', body: '', cc: '', from: '', replyTo: '', subject: '"${env.JOB_NAME} Success (<${env.BUILD_URL}|Open>)"', to: 'lev@gmail.com'
        }
   }
}
