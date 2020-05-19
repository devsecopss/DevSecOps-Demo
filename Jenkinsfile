// Starting the pipeline
pipeline {
  agent any
  // using maven from jenkins (name has to be the same as the one in the configuration)
  tools {
    maven 'maven'
  }
  
  stages {
    stage ('Initialize') {
      steps {
         sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
        echo "Starting Your Pipeline Build"
       }
      stage ('Cloning Repository') {
        steps {
          git credentialsId: '610370c6-a31f-49a1-9293-772af8d73f41', url: 'https://github.com/devsecopss/DevSecOpsDemo'
          sh 'ls -l'  
         }
        }
    }
    
   // Check for any unintentionally left credentials 
   stage ('GitCheckSecrets') {
      steps {
        sh 'touch trufflehog_results'
        sh 'docker run --rm dxa4481/trufflehog --regex --entropy=False --json https://github.com/devsecopss/webapp-pipeline.git > trufflehog_res.json || true'
        sh 'cat trufflehog_res.json' 
       }
     //double Scan using git leaks
     steps {
       sh 'docker run --rm zricethezav/gitleaks -r=https://github.com/devsecopss/webapp-pipeline  --pretty --verbose > res.json' //by default ouput json
       sh 'cat res.json'
     }  
    }
    // Third Parties Vulnerabilities Analysis
    stage ('Dependencies Analysis') {
      steps {
          sh '''
                dependency-check || true
             '''
          sh 'cat /var/lib/jenkins/workspace/webapp-pipeline/odc-reports/dependency-check-report.xml'
       }
    // Double Check You never know  
      steps {
        sh 'snyk test --json ---show-vulnerable-paths=all > snyk_res.json'
        sh 'snyk wizard'
        sh 'snyk monitor'
       }
    }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
    // Static Code Analysis 
    stage ('SAST') {
        steps {
          withSonarQubeEnv('sonar') {
            sh 'mvn sonar:sonar'
            sh 'cat target/sonar/report-task.txt'
          }
        }
      }
     stage("Quality Gate"){
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
          }
      }

    // Artifact Repository uploader to Nexus Server
    stage("Artifact Upload"){
      echo "Just like we did in the DevOps demo, but using commands in this jenkinsfile"  
    }
    
    
    
    
    // checki
      stage ('Checking Services Health'){
        steps {
        	echo "Using NMAP to check Service vulnerability"
        	sh 'nmap'
            sh 'echo Add my Script Over Here!!'
          }
      }


    stage ('Deploy-To-Tomcat') {
            steps {
               sshagent(['610d3050-5b62-4edc-8395-acddb916ec5c']) {
                    sh 'scp -o StrictHostKeyChecking=no target/*.war  yacine@webserver:/opt/tomcat/webapps/webapp-pipeline.war'
                   }
            }
   }
    
    stage ('DAST') {
      steps {
         sh ' docker run -t owasp/zap2docker-stable zap-baseline.py -t http://webserver/webapp-pipeline/ || true'
      }
    }
    
    
    stage("Upload reports To Defect Dojo"){
    	echo "Trying to Upload All the report into our Vulnerability management tool Defect Dojo"
    }
    
  }
}
