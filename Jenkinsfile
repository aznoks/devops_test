pipeline {
    agent any
    parameters {
      choice(
          choices: ['dev' , 'master'],
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
                  sh 'mv $WORKSPACE/target/*.jar $WORKSPACE/target/application_${GIT_BRANCH##*/}.jar'
                  sh 'curl -H \'X-JFrog-Art-Api:AKCp5dKPYYsuHmMuSGgXouBiZRgLxgQbRdMEx8tdbJYtY8tRnDhker734SX6V2yuNaXQAnPMc\' -T application_${GIT_BRANCH##*/}.jar http://localhost:8081/artifactory/jenkins-integration/'
                }
            }
        }

        stage ('Deploy from Dev branch'){
            when {
              allof{
                branch "develop"
                expression { params.APP_VERSION == 'dev' }
              }
            }
            steps{
                sh 'pkill -9 -f \'application_\' || true'
                sh 'JENKINS_NODE_COOKIE=dontKillMe nohup java -jar target/application_develop.jar &'
	          }
	      }

        stage ('Check App Availability'){
          when {
            branch "develop"
          }
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

        stage ('Deploy from Master branch'){
          when {
            allof{
              branch "master"
              expression { params.APP_VERSION == 'master' }
            }
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
                sh 'curl -H \'X-JFrog-Art-Api:AKCp5dKPYYsuHmMuSGgXouBiZRgLxgQbRdMEx8tdbJYtY8tRnDhker734SX6V2yuNaXQAnPMc\' -O http://localhost:8081/artifactory/jenkins-integration/application_master.jar'
                sh 'JENKINS_NODE_COOKIE=dontKillMe nohup java -jar application_master.jar &'
            }
        }
    }
}
