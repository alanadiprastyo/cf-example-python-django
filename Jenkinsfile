pipeline {
    agent any

    stages{
        stage('Checkout'){
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/alanadiprastyo/cf-example-python-django.git']]])
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
    }
}
