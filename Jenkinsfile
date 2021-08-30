pipeline {
agent {
   label 'jenkinsNode'
}

environment {
    config = readJSON file: 'config.json'
	
	GitSource = "${config.git_repo.URL}"
	GitCred = "${config.git_repo.Cred}"
	Branch = "${config.git_repo.branch}"
	
	projectName = "${config.projectDetails.Name}"
	projectVersion = "${config.projectDetails.version}"
	reportsPath = "${config.projectDetails.reportsPath}"
	
	projectLanguage = "${config.build.language}"
	languageVersion = "${config.build.version}"
	buildTool = "${config.build.buildTool}"
	
	//For Ant Build
	antBuildFile = "${config.build.antBuildFile}"
	antCompileTarget = "${config.build.antCompileTarget}"
	antUnitTestTarget = "${config.build.antUnitTestTarget}"
	
	//dotNet build
	dotNetSolutionPath = "${config.build.dotNetSolutionPath}"
	dotNetAppType = "${config.build.dotNetAppType}"
	
	
	skipUnitTestCases = "${config.unitTestCases.skipUnitTestCases}"
	pythonUnitTestFileLocation = "${config.unitTestCases.pythonUnitTestFileLocation}"
	
	skipSonarAnalysis = "${config.sonar.skipSonarAnalysis}"
	sonarProjectAuthToken = "${config.sonar.projectAuthToken}"
	sonarProjectKey = "${config.sonar.projectKey}"
	qgSleepTime = "${config.sonar.sleepTime}"
	
	skipZephyrReports = "${config.zephyr.skipZephyrReports}"
	zephyrServerAddress = "${config.zephyr.serverAddress}"
	zephyrProjectKey = "${config.zephyr.projectKey}"
	zephyrReleaseKey = "${config.zephyr.releaseKey}"
	zephyrParserTemplateKey = "${config.zephyr.parserTemplateKey}"
	zephyrCycleDuration = "${config.zephyr.cycleDuration}"
	zephyrCycleKey = "${config.zephyr.cycleKey}"
	zephyrCyclePrefix = "${config.zephyr.cyclePrefix}"
	zephyrCreatePackage = "${config.zephyr.createPackage}"
	zephyrTestReportsPath = "${config.zephyr.resultXmlFilePath}"
	
	skipTrivyScan = "${config.trivy.skipTrivyScan}"
	trivyVersion = "${config.trivy.version}"
	severity = "${config.trivy.severity}"
	
	skipAccuricsReport = "${config.accurics.skipAccuricsReport}"
	accuricsConfigFilesLocation = "${config.accurics.configFilesLocation}"
	helmLocation = "${config.accurics.helmLocation}"
	terraformLocation = "${config.accurics.terrafomLocation}"
	
	skipHelmPush = "${config.helm.skipHelmPush}"
	helmChartLocation = "${config.helm.chartLocation}"
	
	harborURL = "${config.harbor.URL}"
	harborCredID = "${config.harbor.credID}"
	harborRepoName = "${config.harbor.repoName}"
	harborTagURL = "${config.harbor.tagURL}"
	
	skipImagePush = "${config.docker.skipImagePush}"
	dockerFileLocation = "${config.docker.dockerFileLocation}"
	imageName = "${config.docker.imageName}"
	tagName = "${config.docker.tagName}"
	
	skipCodeMerge = "${config.codeMerge.skipCodeMerge}"
	mergeToBranch = "${config.codeMerge.mergeToBranch}"
	
	skipConfigurek8s = "${config.k8sAWS.skipConfigurek8s}" 
	jenkinsNodeSpinnaker = "${config.k8sAWS.jenkinsNodeSpinnaker}"
	awsCred = "${config.k8sAWS.awsCred}" 
	awsRegion = "${config.k8sAWS.awsRegion}" 
	awsIamAuthenticatorURL = "${config.k8sAWS.awsIamAuthenticatorURL}" 
	awsIamAuthenticatorSHA256URL = "${config.k8sAWS.awsIamAuthenticatorSHA256URL}" 
	
	emailToList = "${config.emailIDs.toList}" 
	emailCCList = "${config.emailIDs.ccList}" 
}

stages {
	stage('Build-Application') {
		steps {
			dir("${workspace}"){
				script {
					sh "mkdir ${workspace}/${reportsPath} -p "
					if ("${projectLanguage}".equalsIgnoreCase("Java") && "${buildTool}".equalsIgnoreCase("maven")) {
						def maven_Home = tool 'maven';
						sh "${maven_Home}/usr/bin/mvn clean package"
					}
					if ("${projectLanguage}".equalsIgnoreCase("Java") && "${buildTool}".equalsIgnoreCase("gradle")) {
						sh 'sudo chmod +x ./gradlew'
						sh './gradlew clean build'
					}
					if ("${projectLanguage}".equalsIgnoreCase("Java") && "${buildTool}".equalsIgnoreCase("ant")) {
						sh 'echo runs Ant target compile using the build.xml file in the current directory.'
						def ant_Home = tool 'ant';
						sh "${ant_Home}/bin/ant -f ${antBuildFile} ${antCompileTarget}"
					}
					if ("${projectLanguage}".equalsIgnoreCase("Python")) {
						echo "pip3 must be installed and configured correctly to run the requirements.txt."
						sh 'pip3 install -r requirements.txt && pip3 install coverage'
					}
					if ("${projectLanguage}".equalsIgnoreCase("DotNet")) {
						sh 'dotnet build ${dotNetSolutionPath}'
					}
				}
			}
		}
    }
	stage('Unit-TestCases') {
		when {
			expression {"${skipUnitTestCases}" != 'true'}
		}
		steps {
			dir("${workspace}"){
				script {
					if ("${projectLanguage}".equalsIgnoreCase("Java") && "${buildTool}".equalsIgnoreCase("maven")) {
						def maven_Home = tool 'maven';
						sh "${maven_Home}/bin/mvn test"
						sh "${maven_Home}/bin/mvn surefire-report:report"
						sh "sudo cp ${workspace}/target/site/surefire-report.html ${workspace}/${reportsPath}/reportUnitTest.html"
					}
					if ("${projectLanguage}".equalsIgnoreCase("Java") && "${buildTool}".equalsIgnoreCase("gradle")) {
						sh './gradlew clean test --no-daemon'
                    }
					if ("${projectLanguage}".equalsIgnoreCase("Java") && "${buildTool}".equalsIgnoreCase("ant")) {
						sh 'echo runs Ant target unittest using the build.xml file in the current directory.'
						def ant_Home = tool 'ant';
						sh "${ant_Home}/bin/ant -f ${antBuildFile} ${antUnitTestTarget}"
					}					
					if ("${projectLanguage}".equalsIgnoreCase("Python")) {
						sh "python3 ${pythonUnitTestFileLocation} test"
					}
					if ("${projectLanguage}".equalsIgnoreCase("DotNet")) {
						sh 'dotnet test ${dotNetSolutionPath} --logger trx'
						
					}
				}
			}
		}
    }
	stage('Zephyr-Reports') {
		when {
			expression {"${skipZephyrReports}" != 'true' && "${skipUnitTestCases}" != 'true'}
		}
		steps {
			dir("${workspace}"){
				script {
					zeeReporter createPackage: "${zephyrCreatePackage}", cycleDuration: "${zephyrCycleDuration}", cycleKey: "${zephyrCycleKey}", cyclePrefix: '', parserTemplateKey: "${zephyrParserTemplateKey}", projectKey: "${zephyrProjectKey}", releaseKey: "${zephyrReleaseKey}", resultXmlFilePath: "${zephyrTestReportsPath}", serverAddress: "${zephyrServerAddress}"
					//zeeReporter createPackage: false, cycleDuration: '30 days', cycleKey: '2', cyclePrefix: '', parserTemplateKey: '1', projectKey: '1', releaseKey: '1', resultXmlFilePath: '', serverAddress: 'https://unisys.zephyrdemo.com'
				}
			}
		}
	}
 	stage('SonarAnalysis-And-QualityGate') {
		when {
			expression {"${skipSonarAnalysis}" != 'true'}
		}
		steps {
			script {
				dir("${workspace}"){
					withSonarQubeEnv('sonarqube'){
						withCredentials([string(credentialsId: "${sonarProjectAuthToken}", variable: 'sonarUsrToken')]) {
							if ("${projectLanguage}".equalsIgnoreCase("Java") || "${projectLanguage}".equalsIgnoreCase("Python")) {
								def scanner_Home = tool 'sonarqube-scanner';
								sh "${scanner_Home}/bin/sonar-scanner -Dsonar.login=${sonarUsrToken} -Dsonar.projectKey=${sonarProjectKey} -Dsonar.java.binaries=."
								sh "echo Please refer sonarscanner usage for any clarifications: https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/"
							
							}
							if ("${projectLanguage}".equalsIgnoreCase("DotNet") && "${dotNetAppType}".equalsIgnoreCase("DotNetCore")) {
								def scanner_Home = tool 'sonar-scanner-msbuild';
								sh "below steps for .net core sonar scanner"
								sh "dotnet ${scanner_Home}/SonarScanner.MSBuild.dll begin /k:${sonarProjectKey} /d:sonar.login=${sonarUsrToken}"
								sh 'dotnet build ${dotNetSolutionPath}'
								sh "dotnet ${scanner_Home}/SonarScanner.MSBuild.dll end /d:sonar.login=${sonarUsrToken}"
								sh "echo Please refer sonarscannermsbuild usage for any clarifications: https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-msbuild/"
							}
						}	
					}
					//Get QualityGate status:
					sleep(time: "${qgSleepTime}", unit: 'MINUTES')
					def qg = waitForQualityGate()
					if (qg.status != 'OK'){
						sh 'echo If Pipeline fails due to "PENDING" or "INPROGRESS" please increase the time based on the report upload time.'
						error "Pipeline aborted due to quality gate failure: ${qg.status}"
					}
				}
			}
		}
	}
	stage('Docker-Build') {
		steps {
			dir("${workspace}"){
				script {
					sh "docker build -f $workspace/${dockerFileLocation} -t ${imageName} ."
				}
			}	
		}
    }
	stage('Trivy-Scan') {
		when {
			expression {"${skipTrivyScan}" != 'true' }
		}
		steps {
			dir("${workspace}"){
				script {
					sh "echo scanning application image"
					sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:${trivyVersion} -q image --exit-code 1 --no-progress -s '${severity}' ${imageName}"
				}
			}
		}
    }
	stage('Image-Push-To-Harbor') {
		when {
			expression {"${skipImagePush}" != 'true' }
		}
		steps {
			dir("${workspace}"){
				withDockerRegistry([ url: "${harborURL}", credentialsId:"${harborCredID}"]) {
					script{
						sh 'echo harbor certificate need to be placed under /usr/local/share/ca-certificates in Jenkins node'  
						sh 'docker tag ${imageName}:latest ${harborTagURL}/${harborRepoName}/${imageName}:$BUILD_NUMBER'
						sh 'docker push ${harborTagURL}/${harborRepoName}/${imageName}:$BUILD_NUMBER'
					}
				}
			}
		}
	}
	stage('Accurics-Reports') {
		when {
			expression {"${skipAccuricsReport}" != 'true'}
		}
		steps {
			dir("${workspace}"){
				script {
					sh "sudo cp ${workspace}/${accuricsConfigFilesLocation}/accurics /usr/local/bin/"
					sh 'chmod +x /usr/local/bin/accurics'
					sh 'export PATH=$PATH:/usr/local/bin/'
					if (env.terraformLocation != ' ') {
						dir("${workspace}/${terraformLocation}") {
							sh "sudo rm -rf config"
							sh "sudo cp -f ${workspace}/${accuricsConfigFilesLocation}/config-terraform ${workspace}/${terraformLocation}/config"
							sh "accurics init"
							sh "accurics plan"
							sh "sudo cp accurics_report.html ${workspace}/${reportsPath}/reportTerraform.html"
						}
					}
					if (env.helmLocation != ' ') {
						dir("${workspace}/${helmLocation}") {
							sh "sudo rm -rf config"
							sh "cp ${workspace}/${accuricsConfigFilesLocation}/config-helm ${workspace}/${helmLocation}/config"
							sh "accurics scan helm"
							sh "sudo cp accurics_report.html ${workspace}/${reportsPath}/reportHelm.html"
							sh "sudo rm -rf config"
							sh "cp ${workspace}/${accuricsConfigFilesLocation}/config-kubernetes ${workspace}/${helmLocation}/config"
							sh "accurics scan k8s"
							sh "sudo cp accurics_report.html ${workspace}/${reportsPath}/reportk8s.html"
						}
					}	
				}
			}
		}
    }
	stage('Helm-Push-To-Harbor') {
		when {
			expression {"${skipHelmPush}" != 'true'}
		}
		steps {
			dir("${workspace}/${helmChartLocation}"){
				script {
					withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${harborCredID}", usernameVariable: 'username', passwordVariable: 'password']]) {
						sh('''
						helm repo add ${harborRepoName} ${harborURL}/chartrepo/${harborRepoName} --username $username --password $password --force-update
						helm repo update
						helm package .
						echo 'pushing helm to harbor'
						helm push *.tgz ${harborRepoName} --version=${tagName}
						''')
					}
				}
			}	
		}
	}
	stage('Merge-Code-To-Release-Branch') {
	    when {
			expression {"${skipCodeMerge}" != 'true'}
		}
		steps {
			dir("${workspace}"){
				script {
					withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${GitCred}", usernameVariable: 'username', passwordVariable: 'password']]) {	
						sh('''
						git config --local user.name ${username}
						git config --local user.password ${password}
						git config --global credential.helper store
						git checkout dev
						git push origin dev:"${mergeToBranch}"
						''') 
					}
				}
			}	
		}
	}
}	

post {
	always{
		sh "mkdir /tmp/${reportsPath}/ -p"
	    sh "mkdir /tmp/${reportsPath}/env.JOB_NAME -p"
	    sh "mkdir /tmp/${reportsPath}/${env.JOB_NAME}/${env.BUILD_NUMBER} -p"
		sh "cp ${workspace}/${reportsPath}/  /tmp/${reportsPath}/${env.JOB_NAME}/${env.BUILD_NUMBER} -r"
		echo 'This build Reports are available on the Location: /tmp/${reportsPath}/${env.JOB_NAME}/${env.BUILD_NUMBER}'
		
		emailext (
			to: "${emailToList}",
			subject: "JOB " + env.JOB_NAME + ", BUILD " + currentBuild.displayName + " completed with status: " + currentBuild.currentResult,
			body: '''${SCRIPT, template="groovy-html.template"}''',
			mimeType: 'text/html',
			attachLog: true,
			attachmentsPattern: "${reportsPath}/*"
		)
	}
}
}
