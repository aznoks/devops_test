pipeline {
    agent any
    parameters {
      choice(
          choices: ['dev' , 'prod'],
          description: '',
          name: 'APP_VERSION')
    }

    stages{
        stage('Build'){
            steps {
                sh 'mvn clean install'
            }
            post {
                success {
                  sh 'mv $WORKSPACE/target/*.jar application_${GIT_BRANCH##*/}.jar'
                  sh 'curl -H \'X-JFrog-Art-Api:AKCp5dKPYYsuHmMuSGgXouBiZRgLxgQbRdMEx8tdbJYtY8tRnDhker734SX6V2yuNaXQAnPMc\' -T application_${GIT_BRANCH##*/}.jar http://localhost:8081/artifactory/jenkins-integration/'
                }
            }
        }

        stage ('Deploy from Dev branch'){
            when {
                expression { params.APP_VERSION == 'dev' }
            }
            steps{
                sh 'pkill -9 -f \'application_\' || true'
                sh 'JENKINS_NODE_COOKIE=dontKillMe nohup java -jar application_develop.jar &'
	          }
	      }

        stage ('Check App Availability'){
          steps{
              script{
                  try {
                    sh "curl -s --head  --request GET  localhost:8080/up | grep '200'"
                    checkValue = true
                  }
                  catch (Exception e) {
                    checkValue = false
                  }
              }
          }
        }

        stage ('Deploy from Prod branch'){
            when {
                expression { params.APP_VERSION == 'prod' }
            }
            steps {
                sh 'pkill -9 -f \'application_\' || true'
                sh 'JENKINS_NODE_COOKIE=dontKillMe nohup java -jar application_master.jar &'
            }
        }

        stage ('Update to prod when dev is unhealthy'){
            when {
                expression {
                  !checkValue
                }
            }
            steps {
                sh 'pkill -9 -f \'application_\' || true'
                sh 'JENKINS_NODE_COOKIE=dontKillMe nohup java -jar application_master.jar &'
            }
        }
    }
}
