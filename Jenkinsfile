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
                         sh 'mvn clean package verify -DskipTests -s settings.xml'    
                     }
                }
                stage('Test') {
                    steps {
                        sh 'mvn surefire:test  -s settings.xml'
                    }
                }
                stage('site') {
                    steps {
                        echo 'Noting to do now'
                    }
                }
                stage('Deploy'){
                    steps {
                        echo 'Standalone app not deploying'
                    }
                }
                stage('Stash') {
                    when {
                        expression {
                           return env.BRANCH_NAME ==~ /(?i)(^master$|^release.*)/
                        }
                    }

                    steps {
                           stash name:'artifacts', includes: 'target/dependency/**'

                       }
                }
            }
            
        }
        
        stage('Build docker image and deploy') {
        
            agent any
        
            when {
                expression {
                    return env.BRANCH_NAME ==~ /(?i)(^master$|^release.*)/
                }
            }
            environment {
                appImage = ''
                mavenPomVersion = ''
                isSnapshotVersion = false
                registryUrl = ''
            }

            stages {
                
                stage('Initialize') {
                    steps {
                           unstash name:'artifacts'
                           sh 'ls -lhart'
                       }
                }

                stage('Build Image') {
                    steps {
                    	script {
                            mavenPomVersion = getPomVersion()
                            echo "Maven version is ${mavenPomVersion}"
                            echo 'Creating docker image'
                            isSnapshotVersion = mavenPomVersion.toLowerCase().contains('snapshot')
                            registryUrl = isSnapshotVersion ? env.DOCKER_SNAPSHOTS_REGISTRY_URL : env.DOCKER_RELEASE_REGISTRY_URL
                            echo 'Building image with target URL ${registryUrl}'
                            def registryPrefix = isSnapshotVersion ? env.DOCKER_SNAPSHOTS_REGISTRY : env.DOCKER_RELEASE_REGISTRY
                            withDockerRegistry(credentialsId: 'Deployer', url: registryUrl) {
				    	        appImage = docker.build("${registryPrefix}/apps/k-ci-sample:${mavenPomVersion.toLowerCase()}")
						    }
	                    }
                    }
                }
                stage('Deploy Image') {
       				steps {
       					script {
							echo 'Publishing docker image'
       				        withDockerRegistry(credentialsId: 'Deployer', url: registryUrl) {
				    	        appImage.push()
				    	        appImage.push('latest')
						    }           
					    }
				    }
                }
            }

            post {
                success {
                    echo 'Cleaning up...'
                    cleanWs()
                }
                aborted {
                    echo 'Cleaning up...'
                    cleanWs()
                }

            }

        }

        
    }
    
    post {
        always {
            script {
                def parentProject = env.JOB_NAME.replace(env.JOB_BASE_NAME, "")
                def buildURL = env.JENKINS_URL
                def newBuildURL = "${buildURL}blue/organizations/jenkins/${parentProject}/detail/${env.BRANCH_NAME}/${env.BUILD_NUMBER}/pipeline"
                emailext body: """${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - ${currentBuild.currentResult}: Check console output at ${newBuildURL} to view the results.""",
                    recipientProviders: [developers(), requestor(), brokenTestsSuspects(), brokenBuildSuspects(), culprits()], 
                    subject: '$PROJECT_NAME - Build#$BUILD_NUMBER - $BUILD_STATUS!'
            }
        }

    }
}

def printEnv() {
    sh 'env'        
}


def getPomVersion() {
	def pVersion = readMavenPom().getVersion()
	if(pVersion?.trim()){
		return pVersion.trim()
    } else {
        return pVersion.getParent().getVersion().trim()
	}    
}

