pipeline {

	agent none
    
    stages {
        stage('Maven Build and Package') {
           
           agent {
               
               docker {
                   image "maven"
               }
           }

			stages {
				stage('Initialize') {
					steps {
               			printEnv()
               			sh 'mvn -v'
               			sh 'ls -lhart'
           			}	   
				}
				stage('Checkout') {
				    steps {
    				    checkout scm
    				    sh 'git branch -vv'
    				}
				}
			    stage('Build') {
			    	steps {
	     				sh 'mvn clean package verify -DksipTests'    
	     			}
        		}
        		stage('Test') {
        			steps {
			            sh 'mvn surefire:test'
        			}

				}
        	}
        }
    }

}

def printEnv() {
	sh 'env'        
}
