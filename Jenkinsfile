//deklarasi variable
def imageLineDev = ""
pipeline {
    agent any
   
    stages{
        stage('Checkout'){
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/alanadiprastyo/cf-example-python-django.git']]])
            }
        }
        stage('SCA test safety'){
            steps{
                sh "docker run -v $(pwd):/src --rm hysnsec/safety check -r requirements.txt --json > sca-scaning-safety.json"
	    archiveArtifacts artifacts: 'sca-scaning-safety.json', followSymlinks: false
            }
        }
      stage('Code Quality Check via SonarQube') {
   	steps {
       		script {
		   sh '''/var/lib/jenkins/sonar-scanner/bin/sonar-scanner \
		   -Dsonar.projectKey=demo-devsecops \
		   -Dsonar.sources=. \
		   -Dsonar.css.node=. \
		   -Dsonar.host.url=http://192.168.1.31:9000\
		   -Dsonar.login=44c1e369a8e63f387d0935b95e1f4ddb018ec593'''
              		}
       		}
	}
        stage('Unit Test'){
            steps{
                sh '''
		python -m unittest composeexample.utils
                '''
            }
        }
        stage('Build Docker Image'){
            steps{
                sh '''
                docker build -t demo-django .
		docker tag demo-django quay.io/alanadiprastyo/demo-django:latest
                '''
            }
        }
        stage('Push Docker Image'){
            steps{
                sh '''
                docker push quay.io/alanadiprastyo/demo-django:latest
                '''
            }
        }
        stage('Scan Image use anchore'){
		steps{	
			sh "echo 'scan image docker use ancore'"
			script {
			imageLineDev = 'quay.io/alanadiprastyo/demo-django'
			writeFile file: 'anchore_images', text: imageLineDev
			anchore name: 'anchore_images'
			}
		}
        }
    }
}
