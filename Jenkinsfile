#!groovy

pipeline {
agent {
        label 'docker'
    }
	stages {
		stage('Git Clone') {
			steps {
			    checkout scmGit(branches: [[name: '*/email-notification']], extensions: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/DanielAvidan/trainmefordevsecops-1.git']])
			}
		}
        stage('SAST') {
            environment {
                SONAR_TOKEN = credentials('sonarqube')
                SONAR_SCANNER_VERSION = '4.7.0.2747'
                SONAR_SCANNER_HOME = "$HOME/.sonar/sonar-scanner-$SONAR_SCANNER_VERSION-linux"
                PATH = "$SONAR_SCANNER_HOME/bin:$PATH"
                SONAR_SCANNER_OPTS = '-server'
            }
            steps {
                sh 'curl --create-dirs -sSLo $HOME/.sonar/sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-$SONAR_SCANNER_VERSION-linux.zip'
                sh 'unzip -o $HOME/.sonar/sonar-scanner.zip -d $HOME/.sonar/'
                sh '''sonar-scanner \
                    -Dsonar.organization=danielavidan \
                    -Dsonar.projectKey=trainmefordevsecops-1\
                    -Dsonar.sources=. \
                    -Dsonar.host.url=https://sonarcloud.io'''
            }
        }
  
        stage('Build and Tag'){
            environment{
                dockerhub=credentials('dockerhub')
            }
            steps{
                sh 'ls'
				sh 'pwd'
				sh 'docker build -t snake .'
                sh 'docker tag snake davidan52/snake:daniel.2'
                
            }
        }
        stage('image vulnerability test'){
            steps{
                sh 'trivy davidan52/snake:daniel.2'
            }
        }
        stage('post-to-dockerhub'){
            steps{
               sh 'docker push davidan52/snake:daniel.2' 
            }
        }
        stage('pull-image-server'){
            steps{
                sh 'docker compose down'
                sh 'docker compose up'
            }
        }
	    stage('DAST'){
	        steps{
	            script {
            def zap = tool name: 'OWASP ZAP', type: 'org.jenkinsci.plugins.tools.ToolInstallation'
            sh "${zap}/zap.sh -cmd -cmdline '-addonupdate -quickprogress -config api.disablekey=true -config api.addrs.addr.name=.* -config api.addrs.addr.regex=true'"
            sh "${zap}/zap.sh -cmd -cmdline '-quickurl http://3.253.131.62:8080 -quickscan -scanners all'"
        }
	        }
	    }
    }
}