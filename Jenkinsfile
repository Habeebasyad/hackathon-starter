pipeline {
    agent any
     options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '5', numToKeepStr: '3')
    environment{
	    dockerregistry="registry.hub.docker.com/"
        dockerrepo="habi9/habeeba_19"
        dockerimage=""
		kubernetesfile="kube-deployment_dockerprivate.yaml"
	//Checkout Code stage	
    stages {
        stage('checkout') {
            steps {
                git branch: 'development', credentialsId: '2ea143eb-76aa-488a-8239-561e3baf6c6a', url: 'https://github.com/Habeebasyad/hackathon-starter.git'
                
            }
        }
   

//Build
//ignoring artifact upload
stage("BuildCode"){
steps{
nodejs(nodeJSInstallationName: 'nodejs18.6.0'){
sh "npm install"           
            }
        }
}
 stage("ExcuteSonarQubeReport")
{
nodejs(nodeJSInstallationName: 'nodejs18.6.0')
sh "npm run sonar"
}       
   //create docker image out of new code
        stage("BuildDockerImage"){
            steps{
                script{
                    dockerimage = docker.build("${env.dockerregistry}${env.dockerrepo}:$BUILD_NUMBER", "-f Dockerfile .")
                }
            }
        }
        
        //push image to docker private registry, 'dockerprivatecreds' are configured in jenkins credential store as username and password
        stage("PushToDockerPrivateRegistry"){
		    steps{
				script{
                    docker.withRegistry("https://${env.dockerregistry}","dockerprivatecreds"){
                        dockerimage.push()
                    }
                
				}
			}
		}

        //update version with build number in kubernetes deployment using sed command
        stage("UpdateTagVersion"){
		    steps{
				sh "sed -i '/${env.tagplaceholder}/ s/${env.tagplaceholder}/$BUILD_NUMBER/' ${env.kubernetesfile}"
			}
		}
		
		
		//Deploy to cluster
		stage("DeployPodsToCluster"){
		   steps{
		       sh "kubectl apply -f ${env.kubernetesfile}"
		   }
		}
		
		//wait for deployment status using rollout commad
		stage("WaitForK8Status"){
		   steps{
		      script{
		           depstatus= sh(returnStatus: true, script: "kubectl rollout status deployment ${env.deploymentName} -n ${env.defaultNamespace}")
			       if(depstatus == 0){
					  currentBuild.result = "SUCCESS"
				   }
				   else{
				       currentBuild.result = "FAILURE"
				   }  

			  } 
		   }
		}		
    }
    }
        }
    }
}
