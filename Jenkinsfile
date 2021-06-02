pipeline {
    agent any
	tools
	{
		maven 'mavenhome'
		jdk 'jdkhome'
	}
	parameters {
            string(name: 'Version', defaultValue: '8.0.0')
        }
    stages {
		stage('checout scm')
		{
			steps
			{
				git branch: 'master', url: 'https://github.com/nag449/PetClinic.git'
			}
		}
        stage('build') {
           steps {
              script { 

                 def server = Artifactory.server 'artifactory'
				 def rtMaven = Artifactory.newMavenBuild()
                 rtMaven.deployer server: server, releaseRepo: 'nallangi-repo', snapshotRepo: 'nallangi-repo'
				 rtMaven.deployer.deployArtifacts = false
				 
				 def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install -Dbuild.number=$Version'
				 
				 echo "sleeping 10 secs"
				 sleep 10
				 rtMaven.deployer.deployArtifacts buildInfo
               }
            }
        }
		
		
    } 
}
