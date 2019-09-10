def mvnHome
def remote = [:]
    	remote.name = 'deploy'
    	remote.host = '192.168.33.15'
    	remote.user = 'root'
    	remote.password = 'vagrant'
	remote.host = '192.168.56.65'
    	remote.user = 'ansible'
    	remote.password = 'ansible'
    	remote.allowAnyHosts = true
pipeline {
    
	agent none
	
	stages {
		//def mvnHome
		stage ('Preparation') {
		    agent {
		        label 'slave'
		    }
		    steps {
			    git 'https://github.com/yaswanth4386/Maven-Java-Project.git'
			    stash 'Source'
			    script{
			        mvnHome = tool 'maven3.6'
			    }
		    }
		}
		stage ('Static Analysis'){
			agent {
				label "slave"
            }
			steps {
				sh "'${mvnHome}/bin/mvn' clean cobertura:cobertura"			
			}
			post {
                success {
                    cobertura autoUpdateHealth: false, autoUpdateStability: false, coberturaReportFile: 'target/site/cobertura/coverage.xml', conditionalCoverageTargets: '70, 0, 0', failUnhealthy: false, failUnstable: false, lineCoverageTargets: '80, 0, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '80, 0, 0', onlyStable: false, sourceEncoding: 'ASCII', zoomCoverageChart: false
                }
            }
		}
		stage ('build'){
			agent {
				label "slave"
            }
			steps {
				sh "'${mvnHome}/bin/mvn' clean package"			
			}
			post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    archiveArtifacts '**/*.war'
                    fingerprint '**/*.war'
                }
            }
		}
		stage('Deploy-to-Stage') {
		     agent {
		        label 'slave'
		    }
		    //SSH-Steps-Plugin should be installed
		    //SCP-Publisher Plugin (Optional)
		    steps {
		        //sshScript remote: remote, script: "abc.sh"  	
			sshPut remote: remote, from: 'target/java-maven-1.0-SNAPSHOT.war', into: '/root/tomcat9/webapps/webapp1.war'		        
		    }
    	}
    	stage ('Integration-Test') {
			agent {
				label "slave"
            }
			steps {
				parallel (
					'integration': { 
						unstash 'Source'
						sh "'${mvnHome}/bin/mvn' clean verify"
      							  						
					}, 'quality': {
						unstash 'Source'
						sh "'${mvnHome}/bin/mvn' clean test"
					}
				)
			}
		}
		stage ('approve') {
			agent {
				label "slave"
            }
			steps {
				timeout(time: 7, unit: 'DAYS') {
					input message: 'Do you want to deploy?', submitter: 'admin'
				}
			}
		}
		stage ('Prod-Deploy') {
			agent {
				label "slave"
            }
			steps {
				unstash 'Source'
				sh "'${mvnHome}/bin/mvn' clean deploy"	
				//sshPut remote: remote, from: 'target/java-maven-1.0-SNAPSHOT.war', into: '/home/ansible/ansible-files/ansibleRoles/tomcat/files/webapp1.war'		        
				sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible-controller', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''cd ansible-files
git pull origin master
ansible-playbook ansibleRoles/tomcat.yml''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'ansible-files/ansibleRoles/tomcat/files', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '**/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
			}
			post {
				always {
					archiveArtifacts '**/*.war'
				}
				//steps {
				//	sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible-controller', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''cd ansible-files
//git pull origin master
//ansible-playbook ansibleRoles/tomcat.yml''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'ansible-files/ansibleRoles/tomcat/files', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '**/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
//				}			
			}
		}
    	
	}	
}
