pipeline {
    agent any

    stages{
        stage('Checkout'){
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/alanadiprastyo/jaguar-java-1.git']]])
            }
        }
        stage('Build Maven'){
            steps{
                sh '''
                export MAVEN_HOME=/opt/maven
                export PATH=$PATH:$MAVEN_HOME/bin
                mvn clean install
                '''
            }
        }
        stage('Build Docker Image'){
            steps{
                sh '''
                export MAVEN_HOME=/opt/maven
                export PATH=$PATH:$MAVEN_HOME/bin
                mvn docker:build
                '''
            }
        }
        stage('Push Docker Image'){
            steps{
                sh '''
                docker push registry.lab-home.example.com/jaguar-java:latest
                '''
            }
        }
        stage('Deploy ke Dev'){
            steps{
                sh '''
		oc delete -f dc/dc-dev.yaml -n development
                oc apply -f dc/dc-dev.yaml -n development
                '''
            }
        }
	stage('Approve Image ke Testing?'){
            input {
                message "Apakah Image dev akan di tag ke Testing?"
                ok "Yes, Tag Image ke Testing"
            }
	   steps{
		sh '''
		docker tag registry.lab-home.example.com/jaguar-java:latest registry.lab-home.example.com/jaguar-java:testing
		docker push registry.lab-home.example.com/jaguar-java:testing
		'''
	   }
	  post{
                success {
                    echo 'Image Testing Berhasil di simpan ke registry '
                }
                failure {
                    echo 'Image Testing Gagal di simpan ke registry'
                }
	  }
	}
    }
}
