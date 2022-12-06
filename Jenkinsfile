//@Library('t24-jenkins-shared-libraries') _


/////// ******************************* Code for fectching Failed Stage Name ******************************* ///////
// import io.jenkins.blueocean.rest.impl.pipeline.PipelineNodeGraphVisitor
// import io.jenkins.blueocean.rest.impl.pipeline.FlowNodeWrapper
// import org.jenkinsci.plugins.workflow.support.steps.build.RunWrapper
// import org.jenkinsci.plugins.workflow.actions.ErrorAction

// Get information about all stages, including the failure cases
// Returns a list of maps: [[id, failedStageName, result, errors]]
// @NonCPS
// List<Map> getStageResults( RunWrapper build ) {

//     // Get all pipeline nodes that represent stages
//     def visitor = new PipelineNodeGraphVisitor( build.rawBuild )
//     def stages = visitor.pipelineNodes.findAll{ it.type == FlowNodeWrapper.NodeType.STAGE }

//     return stages.collect{ stage ->

//         // Get all the errors from the stage
//         def errorActions = stage.getPipelineActions( ErrorAction )
//         def errors = errorActions?.collect{ it.error }.unique()

//         return [ 
//             id: stage.id, 
//             failedStageName: stage.displayName, 
//             result: "${stage.status.result}",
//             errors: errors
//         ]
//     }
// }

// Get information of all failed stages
// @NonCPS
// List<Map> getFailedStages( RunWrapper build ) {
//     return getStageResults( build ).findAll{ it.result == 'FAILURE' }
// }

/////// ******************************* Code for fectching Failed Stage Name ******************************* ///////

pipeline {
  agent any

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "docker4028/app:${GIT_COMMIT}"
    applicationURL="http://java-spring-devsecops.eastus.cloudapp.azure.com"
    applicationURI="/increment/99"
  }

  stages {

     stage('Build Artifact - Maven') {
       steps {
         withEnv(["PATH+MAVEN=${tool 'maven-coe'}/bin"] ) {
				sh "mvn clean package -DskipTests=true"
        }
         archiveArtifacts 'target/*.jar'
       }
     }

    //  stage('Unit Tests - JUnit and JaCoCo') {
    //    steps {
    //      withEnv(["PATH+MAVEN=${tool 'maven-coe'}/bin"] ) {
		// 		sh "mvn test"
    //     }
    //    }
    //  }

    //  stage('Mutation Tests - PIT') {
    //    steps {
    //      withEnv(["PATH+MAVEN=${tool 'maven-coe'}/bin"] ) {
		// 		sh "mvn org.pitest:pitest-maven:mutationCoverage"
    //     }
    //    }
    //  }

     /*stage('SonarQube - SAST') {
       steps {
         withSonarQubeEnv('SonarQube') {
           withEnv(["PATH+MAVEN=${tool 'maven-coe'}/bin"] ) {
		      sh "mvn sonar:sonar \
	 	              -Dsonar.projectKey=numeric-application \
	 	              -Dsonar.host.url=http://devopsjenkins-lb-1194555385.us-east-1.elb.amazonaws.com:81"
          }
        }
         timeout(time: 2, unit: 'MINUTES') {
           script {
             waitForQualityGate abortPipeline: true
           }
         }
       }
     }*/

	//  stage('Vulnerability Scan - Docker') {
  //     steps {
  //        parallel(
  //        	/*"Dependency Scan": {
  //               withEnv(["PATH+MAVEN=${tool 'maven-coe'}/bin"] ) {
	// 			  sh "mvn dependency-check:check"
  //               }
	//  		},*/
	//  		"Trivy Scan":{
	//  			sh "bash trivy-docker-image-scan.sh"
	//  		},
	//  		"OPA Conftest":{
	//  			sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
	//  		}   	
  //     	)
  //      }
  //    }
    

     stage('Docker Build and Push') {
       steps {
         withDockerRegistry([credentialsId: "dockerhubcreds", url: ""]) {
           sh 'printenv'
           sh 'docker build -t docker4028/numeric-app:""$GIT_COMMIT"" .'
           sh 'docker push docker4028/numeric-app:""$GIT_COMMIT""'
         }
       }
     }

     stage('Installing helm chart') {
        steps{
          sh 'printenv'
          sh 'sudo helm install demoapp custom_helm_chart --dry-run'
          sh 'sudo helm install demoapp custom_helm_chart'
        }
     }

    //  stage('Vulnerability Scan - Kubernetes') {
    //    steps {
    //      parallel(
    //        "OPA Scan": {
    //          sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
    //        },
    //        "Kubesec Scan": {
    //          sh "bash kubesec-scan.sh"
    //        },
    //        /*"Trivy Scan": {
    //          sh "bash trivy-k8s-scan.sh"
    //        }*/
    //      )
    //    }
    //  }

    //  stage('K8S Deployment - DEV') {
    //    steps {
    //      parallel(
    //        "Deployment": {
    //          withKubeConfig([credentialsId: 'kubeconfig']) {
    //            sh "bash k8s-deployment.sh"
    //          }
    //        },
    //        "Rollout Status": {
    //          withKubeConfig([credentialsId: 'kubeconfig']) {
    //            sh "bash k8s-deployment-rollout-status.sh"
    //          }
    //        }
    //      )
    //    }
    //  }

    //  stage('Integration Tests - DEV') {
    //    steps {
    //      script {
    //        try {
    //          withKubeConfig([credentialsId: 'kubeconfig']) {
    //            sh "bash integration-test.sh"
    //          }
    //        } catch (e) {
    //          withKubeConfig([credentialsId: 'kubeconfig']) {
    //            sh "kubectl -n default rollout undo deploy ${deploymentName}"
    //          }
    //          throw e
    //        }
    //      }
    //    }
    //  }

    // stage('OWASP ZAP - DAST') {
    //    steps {
    //      withKubeConfig([credentialsId: 'kubeconfig']) {
    //        sh 'bash zap.sh'
    //      }
    //    }
    //  }
   
  //     stage('Testing Slack - 1') {
  //     steps {
  //         sh 'exit 0'
  //     }
  //   }

  //  stage('Testing Slack - Error Stage') {
  //     steps {
  //         sh 'exit 0'
  //     }
  //   }

  }

  // post { 
  //       always { 
  //          junit 'target/surefire-reports/*.xml'
  //          jacoco execPattern: 'target/jacoco.exec'
  //          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
  //          dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
  //          publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP ZAP HTML Report', reportTitles: 'OWASP ZAP HTML Report'])
        
 	// 	  // Use sendNotifications.groovy from shared library and provide current build result as parameter 
  //            sendNotification currentBuild.result
  //        }

  //       success {
  //       	script {
	// 	        /* Use slackNotifier.groovy from shared library and provide current build result as parameter */  
	// 	        env.failedStage = "none"
	// 	        env.emoji = ":white_check_mark: :tada: :thumbsup_all:" 
	// 	        sendNotification currentBuild.result
	// 	      }
  //       }

	//     failure {
	//     	script {
	// 		  //Fetch information about  failed stage
	// 	      def failedStages = getFailedStages( currentBuild )
	//           env.failedStage = failedStages.failedStageName
	//           env.emoji = ":x: :red_circle: :sos:"
	// 	      sendNotification currentBuild.result
	// 	    }	
	//     }
  //   }

}