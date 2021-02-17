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
                sh '''
		safety check -r requirements.txt --json | tee safety_output.json
                '''
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
