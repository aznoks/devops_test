pipeline {
    agent any
    stages{
        stage('Build'){
            steps {
                sh 'mvn clean install'
            }
            post {
                success {
                  sh 'mv $WORKSPACE/target/*.jar application_${GIT_BRANCH##*/}.jar'
                  sh 'pkill -9 -f \'application_\' || true'
                  sh 'curl -H \'X-JFrog-Art-Api:AKCp5dKPYYsuHmMuSGgXouBiZRgLxgQbRdMEx8tdbJYtY8tRnDhker734SX6V2yuNaXQAnPMc\' -T application_${GIT_BRANCH##*/}.jar http://localhost:8081/artifactory/application/'
                }
            }
        }
        stage ('Deploy App'){
            steps{
                sh 'JENKINS_NODE_COOKIE=dontKillMe nohup java -jar application_${GIT_BRANCH##*/}.jar &'
	          }
	      }
    }
}
