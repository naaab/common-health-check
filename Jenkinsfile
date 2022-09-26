pipeline {
    agent any
    environment {
        BUILD_VERSION = "1.0.${currentBuild.number}-SNAPSHOT"
    }
    stages{
        
        stage('Create Release Branch') {
            steps {
                echo "Starting Create Release Branch..."
                sh "git checkout -b '${env.BUILD_VERSION}'"
                sh "/var/jenkins_home/apache-maven-3.8.6/bin/mvn versions:set -DnewVersion='${env.BUILD_VERSION}'"
                echo "Create Release Branch: ${currentBuild.currentResult}"
            }
            post {
                success {
                    echo "...Create Release Branch Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                } 
                unsuccessful {
                    echo "...Create Release Branch Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
            }
        }
   
        stage('Build and Test') {
            steps {
                echo "Starting Build and Test..."
                sh "/var/jenkins_home/apache-maven-3.8.6/bin/mvn -Dmaven.test.failure.ignore clean verify"
                echo "Build and Test: ${currentBuild.currentResult}"
            }
            post {
                success {
                    echo "...Build and Test Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                } 
                unsuccessful {
                    echo "...Build and Test Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                    script {
        	            dropLocalReleaseBranch()
                    }
                }
            }
        }
        
        stage('Push Release Branch') {
            steps {
                script {
                    echo "Starting Push Release Branch..."
                    // Success
                    sh "git config --global user.email nabraj@gmail.com"
                    sh "git config --global user.name jenkins"
                    sh "git add pom.xml"
                    sh "git remote rm origin"
                    
                    sh 'git commit -m "Committing Branch"'                    
                    sh "git remote add origin https://github.com/naaab/github-action-mule.git"                    

                    
                    sh "git push --set-upstream origin '${env.BUILD_VERSION}'"
                    echo "Build Successful...branch ${env.BUILD_VERSION} committed"
                } 
            }
            post {
                success {
                    echo "...Push Release Branch Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                } 
                unsuccessful {
                    echo "...Push Release Branch Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                    script {
        	            dropLocalReleaseBranch()
                    }
                }
            }
        }
        
        stage('Deploy Artifact') {
            steps {
                script {
                    echo "Starting Deploy Artifact..."
                    configFileProvider([configFile(fileId: '107138b1-fdbe-45c5-8eb3-64a520257c07', targetLocation: 'settings.xml', variable: 'MAVEN_SETTINGS_XML')]) {
                        sh '/var/jenkins_home/apache-maven-3.8.6/bin/mvn -U --batch-mode -s $MAVEN_SETTINGS_XML clean package install deploy'
                    }
                    echo "Artifact Deployed: ${currentBuild.currentResult}"
                }
            }
            post {
                success {
                    echo "...Deploy Artifact Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                } 
                unsuccessful {
                    echo "...Deploy Artifact Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                    script {
        	            dropLocalReleaseBranch()
                        // Drop Remote Release Branch
                    }
                }
            }
        }
        
        stage('Deploy to Development') {
            steps {
                script {
                    echo "Starting Deploy to Development"
                        
                    applicationName = readMavenPom().getArtifactId()
                    echo "applicationName=${applicationName}"
                    
                    props = readProperties(file: '/tmp/Jenkins/esapi/deploy.properties')
                    anypointMuleVersion = props['anypoint.mule.version']
                    anypointMuleEnvironment = props['anypoint.mule.environment']
                    anypointUsername = props['anypoint.username']
                    anypointPassword = props['anypoint.password']
                    cloudhubEnv = props['cloudhub.env']
                    cloudhubRegion = props['cloudhub.region']
                    cloudhubWorkerType = props['cloudhub.workerType']
                    cloudhubWorkers = props['cloudhub.workers']
                    cloudhubEnvClientID = props['cloudhub.env.client_id']
                    cloudhubEnvSecretID = props['cloudhub.env.client_secret']
        
                    echo "anypointMuleVersion=${anypointMuleVersion}"
                    echo "anypointMuleEnvironment=${anypointMuleEnvironment}"
                    echo "anypointUsername=${anypointUsername}"
                    echo "anypointPassword=${anypointPassword}"
                    echo "cloudhubEnv=${cloudhubEnv}"
                    echo "cloudhubRegion=${cloudhubRegion}"
                    echo "cloudhubWorkerType=${cloudhubWorkerType}"
                    echo "cloudhubWorkers=${cloudhubWorkers}"
                    echo "cloudhubEnvClientID=${cloudhubEnvClientID}"
                    echo "cloudhubEnvSecretID=${cloudhubEnvSecretID}"
                        
                    echo "Deploy to Development: ${currentBuild.currentResult}"
                }
                
                configFileProvider([configFile(fileId: '107138b1-fdbe-45c5-8eb3-64a520257c07', targetLocation: 'settings.xml', variable: 'MAVEN_SETTINGS_XML')]) {
                    // Run the maven build
                    sh """ mvn -U --batch-mode -s $MAVEN_SETTINGS_XML \
                        -Dmule.version=${anypointMuleVersion}  \
                        -Danypoint.applicationName=${applicationName} \
                        -Danypoint.mule.environment=${anypointMuleEnvironment} \
                        -Danypoint.username=${anypointUsername} \
                        -Danypoint.password=${anypointPassword} \
                        -Dcloudhub.env=${cloudhubEnv} \
                        -Dcloudhub.region=${cloudhubRegion} \
                        -Dcloudhub.workerType=${cloudhubWorkerType} \
                        -Dcloudhub.workers=${cloudhubWorkers} \
                        -Dcloudhub.env.client_id=${cloudhubEnvClientID} \
                        -Dcloudhub.env.client_secret=${cloudhubEnvSecretID} \
                        clean install mule:deploy -P cloudhub """
                }
            }
            post {
                success {
                    echo "...Deploy to Development Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                } 
                unsuccessful {
                    echo "...Deploy to Development Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                    script {
        	            dropLocalReleaseBranch()
                        // Drop Remote Branch
                        // Undeploy from Artifact Management Repo
                        // Rollback to previous version
                    }
                }
            }
        }
        
    }

   post {
       success {
           echo "All Good: ${env.BUILD_VERSION}"
       }
       unsuccessful {
           echo "Not So Good: ${env.BUILD_VERSION}"
       }      
       always {
           echo "Pipeline result: ${currentBuild.result}"
           echo "Pipeline currentResult: ${currentBuild.currentResult}"
       }
   }
   
}

void dropLocalReleaseBranch() {
    echo "Starting Drop Local Release Branch: ${env.BUILD_VERSION} ..."
    sh "git checkout develop"
    sh "git branch -d '${env.BUILD_VERSION}'"
    echo "...Local Release Branch ${env.BUILD_VERSION} Dropped"
}
