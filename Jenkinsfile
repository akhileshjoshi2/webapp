pipeline {
  agent any 
  tools {
    maven 'Maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    
    stage ('Check-Git-Secrets') {
      steps {
        sh 'whoami'
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/akhileshjoshi2/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
    stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/cehkunal/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
        
      }
    }
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
  
  
  stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@54.91.246.140:/home/ubuntu/prod/apache-tomcat-8.5.58/webapps/webapp.war'
                //sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@54.175.162.201:/home/ubuntu/prod/apache-tomcat-8.5.58/webapps/webapp.war'
              }      
           }       
    }
    
    stage ('DAST') {
      steps {
        sshagent(['tomcat']) {
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@52.90.159.201 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://54.91.246.140:8080/webapp/" || true'
        }
      }
    }
    }
}
