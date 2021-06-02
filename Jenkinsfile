pipeline {
    agent any
	tools
	{
		maven 'mavenhome'
		jdk 'jdkhome'
	}
	environment {
        ARTIFACT_NAME = 'petclinic.war'
        AWS_S3_BUCKET = 'testbucketnallangi'
        AWS_EB_APP_NAME = 'petclinic-qa'
        AWS_EB_ENVIRONMENT = 'Petclinicqa-env'
        AWS_EB_APP_VERSION = "${BUILD_ID}"
    }
	parameters {
            string(name: 'Version', defaultValue: '8.0.0')
        }
    stages {
		stage('checkout scm')
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
		stage('Sanity check') {
            steps {
                input "Does the staging environment look ok?"
            }
        }
		
		stage('deploy') {
           steps {
              script { 
				def server = Artifactory.server 'artifactory'
                 def downloadSpec = """{
                    "files": [{
                       "pattern": "nallangi-repo/**/spring-petclinic-$Version-SNAPSHOT.war",
                       "target": "jfrogartifacts/petclinic.war"
                    }]
                 }"""

                 server.download(downloadSpec) 
               }
			   
			   withAWS(credentials: 'jenkins-test-user', region: 'us-east-1')
                    {
						archiveArtifacts 'target/*.war'
						sh 'aws configure set region us-east-1'
						sh 'aws s3 cp ./jfrogartifacts/org/springframework/samples/spring-petclinic/$Version-SNAPSHOT/petclinic.war s3://$AWS_S3_BUCKET/$ARTIFACT_NAME'
						sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
						sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
                    }
            }
        }			
		
    } 
}
