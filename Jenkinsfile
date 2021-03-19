pipeline {
    agent any
    tools {
    maven 'Maven3.6.3'
    }
    
    stages {	
        stage ('Code Quality'){
            environment {
                scannerhome = tool 'sonarqube'
            }
                steps {
                    withSonarQubeEnv('sonarqube') {
						sh "${scannerHome}/bin/sonar-scanner -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.projectKey=WEBPOC:AVNCommunication -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=sonar"
					}
					sh 'mvn validate -f pom.xml'
				}
		}
        
        stage('Build') {
            steps {
                sh 'mvn compile -f pom.xml'
            }
        }

        stage ('Deploy to Test') {
            steps{
				sh 'mvn package -f pom.xml'
				deploy adapters: [tomcat8(credentialsId: 'Tomcat-QA', path: '', url: 'http://172.31.29.124:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
			}
        }
        
        stage('Store Artifacts') {
            steps{
				sh 'mvn package -f pom.xml'
                rtUpload (
                    serverId: 'artifactory',
                    spec: """{
                            "files": [
                                    {
                                        "pattern": "target/*.war",
                                        "target": "libs-release-local"
                                    }
                                ]
                            }"""
                )
                
                rtPublishBuildInfo (
                    serverId: 'artifactory'
                )
            }
        }
        
        stage('UI Tests') {
            steps{
                sh 'mvn test -f functionaltest/pom.xml'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test', reportTitles: ''])
            }
        }
            
        // stage('Performance Test'){
        //     steps{
        //       blazeMeterTest credentialsId: 'Blazemeter', testId: '9014533.taurus', workspaceId: '756606'
        //     }
        // }
        
        stage('Deploy to Prod'){
            steps{
				sh 'mvn clean install -f pom.xml'
                deploy adapters: [tomcat8(credentialsId: 'Tomcat-QA', path: '', url: 'http://172.31.16.110:8080/')], contextPath: '/ProdWebapp', war: '**/*.war'
            }
        }
        
        stage('Prod-Sanity Test'){
            steps{
                sh 'mvn test -f Acceptancetest/pom.xml'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
            }
        }
            
        stage('Slack Notification - Prod'){
            steps{
                slackSend channel: 'alert', message: '\'Prod Build Successful\''
            }
        }
	}
	
	post {
		always {
			deleteDir()
        }
    }
}
