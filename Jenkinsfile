properties([disableConcurrentBuilds()])

pipeline {
    agent { label 'Slave1' }
	
	options {
		buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr:'10'))
		timestamps()
	}	

    tools {
        git 'Default'
    }

    stages {
        stage('Cleanup docker env') {
            steps {
                sh 'docker ps -q -f status=running | xargs --no-run-if-empty docker stop'
                sh 'docker ps -q -f status=exited | xargs --no-run-if-empty docker rm'
                sh 'docker images -q -f dangling=true | xargs --no-run-if-empty docker rmi'            
            }
        }
        stage('Clone from Github') {
            steps {
                git branch: 'master', url: 'https://github.com/Alexey-rnd/SF_8.git'
            }
        } 
        stage('Build container') {
            steps {
                sh 'docker build -t nginx-p8 .' 
            }
        }
        stage('Run nginx container') {
            steps {
                sh 'docker run --rm -p 9889:80 -d --name nginx-project8 nginx-p8'
            }
        }
        stage('Liveness probe') {
            steps {
                sh '''
                   r=`curl -s localhost:9889/alive | grep "alive"`
                   if [ $r = '' ]; then
                        echo 'Something wrong, error!'
                     else
                        echo 'Liveness passed ok'
                   fi
                   '''
            }
        }
        stage('Check MD5--- wont work') {
            steps {
                sh 'echo md5sum.txt'
            }
        }
        stage('Cleanup...') {
            steps {
                sh 'docker ps -q -f status=running | xargs --no-run-if-empty docker stop'
                sh 'docker ps -q -f status=exited | xargs --no-run-if-empty docker rm'
                sh 'docker images -q -f dangling=true | xargs --no-run-if-empty docker rmi'            
            }
        }
    }
  post {
      failure {
         slackSend channel: 'Jenkins', message: 'Failure of docker build and run', color: 'danger', teamDomain: 'devops-xyu2765', tokenCredentialId: 'sl'
      }
      success {
         slackSend channel: 'Jenkins', message: 'Success of docker build and run', color: 'good', teamDomain: 'devops-xyu2765', tokenCredentialId: 'sl'
      }
  }
}
