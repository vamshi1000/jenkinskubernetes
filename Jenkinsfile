pipeline{
    agent any
    environment {
      DOCKER_TAG = getVersion()
	  aksrg = ""
	  akscls = ""
	  acrname = ""
	  acr_username = ""
	  AZURETENANTID = ""
	  AZURESUBSCRIPTIONID = ""
    }
    stages{
        stage('init'){
            steps{
                checkout scm
            }
        }
        
        stage('Build'){
            steps{
                sh "mvn clean package"
				
                sh "sudo docker build . -t ${acrname}/helloapp:latest"
               withCredentials([usernamePassword(credentialsId: 'acrauth', passwordVariable: 'acrpwd', usernameVariable: 'acruser')]) {
                    sh "sudo docker login ${acrname} -u ${acruser} -p ${acrpwd}"
                }
              
                sh "sudo docker push ${acrname}/helloapp:latest "
            }
        }
        
        stage('Deploy'){
            steps{
			withCredentials([usernamePassword(credentialsId: 'spauth', passwordVariable: 'AZURECLIENTSECRET', usernameVariable: 'AZURECLIENTID')]) {
              ansiblePlaybook credentialsId: '496c76de-ea13-4d0f-a945-fec687b54851', disableHostKeyChecking: true, extras: '-e "DOCKER_TAG=${DOCKER_TAG} aksrg=${aksrg} akscls=${akscls}  AZURECLIENTID=${AZURECLIENTID}  AZURECLIENTSECRET=${AZURECLIENTSECRET}  AZURETENANTID=${AZURETENANTID}  AZURESUBSCRIPTIONID=${AZURESUBSCRIPTIONID}"', installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
			  }
            }
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
