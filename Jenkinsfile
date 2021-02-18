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
		    //script {
			//sh "docker run -v \$(pwd):/src --rm hysnsec/safety check -r requirements.txt"
               		sh "docker run -v \$(pwd):/src --rm hysnsec/safety check -r requirements.txt --json > sca-scaning-safety.json  || true"
		    	archiveArtifacts artifacts: 'sca-scaning-safety.json', onlyIfSuccessful: true //fingerprint: true
			//sh  "/usr/local/bin/safety check -r requirements.txt --json"
		    //}
	}
        }
        stage('SCA test pyt'){
            steps{
		    	echo "SCA test Python Taint"
               		sh "docker run -v \$(pwd):/src --rm vickyrajagopal/python-taint-docker pyt . -j > sca-scaning-pyt.json  || true"
		    	archiveArtifacts artifacts: 'sca-scaning-pyt.json', onlyIfSuccessful: true //fingerprint: true
			
	}
        }
        stage('SAST test Bandit'){
            steps{
		    	echo "SAST test Python Bandit"
               		sh "docker run -v \$(pwd):/src --rm secfigo/bandit:latest bandit . -f json > sca-scaning-bandit.json  || true"
		    	archiveArtifacts artifacts: 'sca-scaning-bandit.json', onlyIfSuccessful: true //fingerprint: true
			
	}
        }
      stage('Scan SonarQube') {
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
                sh "docker build -t demo-django ."
                sh "docker tag demo-django quay.io/alanadiprastyo/demo-django:${env.BUILD_ID}"
		sh "docker tag demo-django quay.io/alanadiprastyo/demo-django:latest"
            }
        }
        stage('Push Docker Image'){
            steps{
                sh "docker push quay.io/alanadiprastyo/demo-django:${env.BUILD_ID}"
		sh "docker push quay.io/alanadiprastyo/demo-django:latest"
            }
        }
        stage('Scan Image use anchore'){
		steps{	
			sh "echo 'scan image docker use ancore'"
			script {
			imageLineDev = 'quay.io/alanadiprastyo/demo-django:latest'
			writeFile file: 'anchore_images', text: imageLineDev
			anchore name: 'anchore_images'
			}
		}
        }
	stage('Deploy to Dev'){
		steps{
			echo "deploy to dev"
		}
	}    
	stage('Deploy to Staging'){
		steps{
			echo "deploy to staging"
		}
	}
	stage('DAST NMAP'){
		steps{
			echo "Scan use NMAP"
			sh "docker run --rm -v \$(pwd):/tmp uzyexe/nmap routecloud.net > nmap_out.txt  || true"
		    	archiveArtifacts artifacts: 'nmap_out.txt', onlyIfSuccessful: true //fingerprint: true
		}
	}
	stage('DAST Nikto'){
		steps{
			echo "Scan use Nikto"
			sh "docker run --rm -v \$(pwd):/tmp alpine/nikto -h routecloud.net > nikto.txt  || true"
		    	archiveArtifacts artifacts: 'nikto.txt', onlyIfSuccessful: true //fingerprint: true
		}
	}
	stage('DAST Owasp Zap'){
		steps{
			echo "Scan use Owasp ZAP"
			sh "docker run --rm -v \$(pwd):/tmp owasp/zap2docker-stable zap-baseline.py -t https://routecloud.net > zap.txt  || true"
			//sh "docker run -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-stable zap-baseline.py     -t https://routecloud.net -g gen.conf -r testreport.html > zap.txt"
		    	archiveArtifacts artifacts: 'zap.txt', onlyIfSuccessful: true //fingerprint: true
		}
	}
	stage('Deploy to Prod'){
		steps{
			echo "Deploy to Prod"
		}
	}
    }
}
