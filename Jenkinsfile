// example pipeline for java maven spring-boot app
pipeline {
    agent {
         docker {
            image 'maven:3.5.4-jdk-8-alpine'
            args '-v $WORKSPACE:/tmp/sbapp -u="root" -w /tmp/sbapp'
        }
    }

    //give a hoot—don't pollute!
    options {
    	   buildDiscarder(logRotator(daysToKeepStr: '5', artifactNumToKeepStr: '10'))
	   disableConcurrentBuilds()
	}

    // a build a day keeps the doctors away..	
    triggers {
  	cron '@daily'
    }
      //environment vars can be used for various purposes
	//if you fork this repo you need to do the following actions: 
	//1. Nothing to do or change to download the code to the Jenkins workspace ( it is automatic )
	//2. Make sure you can access the SONARQUBEHOST
        //   - requires that your on the DXC network or DXC Houston VPN
	//   - proxy on browser using:  http://proxy-ccy.houston.entsvcs.net:8080
	//4. Change the ARTIFACTORYPATH in your project's maven pom.xml file
        //      - Artifactory Repo, URL should be https://artifactory.platformdxc-mg.com/artifactory/<YOUR_MAVEN_REPO_NAME>
	//      - Make sure the Artifactory Repo is of type MAVEN
	//5. Add your projects' ARIFACTORY r/w service account id and password to your project's Jenkins credentials vault
	//6. Update the JENKINSCREDID in the environment section to match the crendentials id used to save the Artifactory credentials in jenkins. 
	//7. Update the Maven pom.xml file with your applications information (e.g. groupId and artifactId) 
	  	
    environment {
        PACKAGE_PATH='sb'
        PACKAGE_VERSION_PREFIX='0.0.1'
	SONARHOST ="http://sonar.platformdxc-sb2.com:9000"	    
	JENKINSCREDID = "central"
	ARTIFACTORYPATH = "https://artifactory.dxc.com/artifactory/aet-maven"
	
	    
    }
 
    //start with a clean workspace
    stages {           	    
	stage('Clean') {             
            steps { 
                sh 'echo build'                        
                sh "mvn -B -DskipTests clean package"
            }
        }
    
	  // unit testing	    
       stage('Unit Test') {
            steps {
                sh 'mvn test'
                sh 'mvn surefire-report:report'
            }
            post {
                always {
                  sh 'echo save unit test results'
                  junit '**/target/surefire-reports/TEST-*.xml'	           
                }
            }
        } 	    
	    
	// static code analysis using sonarqube
        stage('static analysis') {
            steps {
                sh 'echo static analysis'
               
                sh "mvn sonar:sonar -Dsonar.host.url=$SONARHOST" 

            }
	}
       
	    //  if the dependacy checker central servers can be a bit flakey.. 
	    //  so this can cause issues that are outside of the project control
	    //  the current workaround is try to do the build later or comment out the checker..
	    //  note you will also need to update the pom.xml file	
	 stage('Dependency check') {
      	     steps {
               	  sh "mvn --batch-mode dependency-check:check"
      		}
      		post {
          	always {
              	publishHTML(target:[
                   allowMissing: true,
                   alwaysLinkToLastBuild: true,
                   keepAll: true,
                   reportDir: 'target',
                   reportFiles: 'dependency-check-report.html',
                   reportName: "OWASP Dependency Check Report"
              	])
              }
      	   }
  	} // end stage 	   
     

        //saves Jenkins workspace to store test results
        //Archiving artifacts this way is not a substitute for using external artifact repositories such as Artifactory  
        //should be considered only for basic reporting and debugging 
        stage('Save Jenkins Workspace'){
            steps {
                sh 'echo store jenkins workspace'
		sh "mvn site"    
                //archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true 
                //archiveArtifacts artifacts: '**/*'
		archiveArtifacts artifacts: '**/target/site/**/*'    
		archiveArtifacts artifacts: '**/target/dependency-check*.*' 
		archiveArtifacts artifacts: '**/target/jacoco-ut/jacoco.xml' 
                archiveArtifacts artifacts: '**/target/site/surefire-report.html'
            }
        }        
        
	// the resulting package created from running an automated build should always be stored in a binary repository ( Artifactory )
	// with the intial package status of "unstable".  Only after the unstable package passes integration, api and/or functional testing 
	// should the package status be marked as "stable" in Artifactory.
	// only runs on the 'master' branch     
        stage('Store to Artifactory'){
	    //when { branch "master" }
	    when {
		branch 'master'    
  		environment name: 'GIT_URL', value: 'https://github.dxc.com/AET/example-java-sb-maven.git'
	    }	
            steps {
                sh 'echo store to Artifactory'
		//sh 'printenv'
                //store automated build artifacts into Artifactory
                //the Artifactory credentials are stored  within Jenkins credentials
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: env.JENKINSCREDID, usernameVariable: 'A_USERNAME', passwordVariable: 'A_PASSWORD']]){
                    sh "echo '<settings xmlns=\"http://maven.apache.org/SETTINGS/1.0.0\" xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xsi:schemaLocation=\"http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd\"><servers><server><id>$JENKINSCREDID</id><username>$A_USERNAME</username><password>$A_PASSWORD</password><filePermissions>664</filePermissions><directoryPermissions>775</directoryPermissions><configuration></configuration></server></servers></settings>' > settings2.xml"
                    sh "mvn deploy -s settings2.xml"    
                }         
            }
        }
        
	// send a build pipeline messages to MS Teams    
        stage('Send message MS TEAMS'){
            steps {
                   sh "echo send message"
                   //sending messages to MS Teams does not require a special stage
                   //From MS Teams add the enterprise github connector to your channel and follow the direction to add the webhook to github
                   //After this is setup and configured you MS Team channel will receive message from github and jenkins.                  
            }
            //steps
        } 
        // stage
    }
    //stages
     post {
        always {
            echo 'One way or another, I have finished'	
            deleteDir() /* clean up our workspace */
        }
        success {
            echo 'I succeeded!'
        }
        unstable {
            echo 'I am unstable :/'
        }
        failure {
            echo 'I failed :('
        }
        changed {
            echo 'Things were different before...'
        }
    }  
}
//pipeline
