pipeline {
    options { 
        timeout(time: 5, unit: 'MINUTES')
	buildDiscarder(logRotator(numToKeepStr: '5')) 
    }
    agent none

	artifactName='*.jar'
  
  	stages {
		stage('Build') {
		  steps {
			echo 'INFO - Starting Build phase'
			sh 'mvn validate'
			sh 'mvn compile'
			sh 'mvn clean install'
		  }
		}
		checkpoint 'after Build before Test'
		stage('Test') {
		  steps {
			echo 'INFO - Starting Test phase'
			parallel(
				"unit test": {
					echo 'unit tests'
					sh 'mvn test'
					sh 'mvn surefire-report:report'			// generate a maven unit test report using surefire
					// this is the path to your unit test report: your-project/target/site/surefire-report.html
				},
				"integration tests": {
					echo 'integration tests'
					sh 'mvn verify'    						// generate a maven integration test report
				},	
				"other tests": {
					echo 'other tests'
					junit '**/target/surefire-reports/TEST-*.xml'    // generate a junit test report
				}
			)
		  }
		}
		checkpoint 'after Test before Package'
		stage('Package') {
		  steps {
			echo 'INFO - Starting Package phase'
			sh 'mvn clean package'
			stash name: 'artifactName', includes: '*.xml'
			}
		}
		post {				
			success {
				unstash 'artifactName'
				archive "target/${artifactName}"
			}
		checkpoint 'after Package before Deploy'
		stage('Deploy') {
		  steps {
			echo 'INFO - Starting Deploy phase'
			sh 'mvn deploy'
	//		sh """
	//   		rm -rf ${somevalue}/webapps/ROOT                 // remove a directory and files without comfirmation
	//			mv -f ${somevalue}                               // move files from one directory to another without prompting
	//			cp -rf ${war} ${somevalue}/webapps/ROOT.war      // copy files from one location to another without prompting
	//			${somevalue}/bin/startup.sh
	//		"""
				}
			}
		}
	}	
}
