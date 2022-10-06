pipeline {
	agent any
	tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}

	environment {
		registry = "juliamayt/samplewebapp"
		registryCredential = 'dockerhub'
	}

	stages {
	    stage('Fetch code') {
            steps {
               git branch: 'master', url: 'https://github.com/mayumiju/SampleWebApp.git'
            }

	    }

	    stage('Build'){
	        steps{
	           sh 'mvn install -DskipTests'
	        }

	        post {
	           success {
	              echo 'Now Archiving it...'
	              archiveArtifacts artifacts: '**/target/*.war'
	           }
	        }
	    }

	    stage('UNIT TEST') {
            steps{
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis'){
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
                withSonarQubeEnv('sonar'){
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=sampleWebApp \
                    -Dsonar.projectName=sampleWebApp \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ '''
                }
            }
        }
        stage("Quality Gate"){
            steps {
              timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                // Parameter indicates wheter to set pipeline to UNSTABLE if Quality Gates Fails
                // true = set pipeline to UNSTABLE , false = don't
                waitForQualityGate abortPipeline: true
                }
            }
	    }

		stage ('Build App Image'){
			steps{
				script {
					dockerImage = docker.build registry + ":v1"
				}
			}
		}

		stage ('Upload Image'){
			steps{
				script {
					docker.withRegistry('', registryCredential) {
						dockerImage.push("v1")
						dockerImage.push('latest')
					}
				}
			}
		}

		stage ('Remove Unused docker image'){
			steps {
				sh "docker rmi $registry:v1"
			}
		}

		stage ('Kubernetes Deploy'){
		    agent {label 'KOPS'}
		        steps {
		            sh "helm upgrade --install --force samplewebapp-stack helm/samplewebappcharts --namespace prod"
		        }
		}
	}
}