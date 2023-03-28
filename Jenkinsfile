pipeline{
    agent any
    environment {
      DOCKER_TAG = getVersion()
	  aksrg = "Demo"
	  akscls = "kubernates142"
	  acrname = "acrdemo142.azurecr.io"
	  acr_username = "acrdemo142"
	  AZURETENANTID = "76a2ae5a-9f00-4f6b-95ed-5d33d77c4d61"
	  AZURESUBSCRIPTIONID = "6003c517-c410-4a40-b9c5-2e288251125e"
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
              ansiblePlaybook credentialsId: 'ansibleserver', disableHostKeyChecking: true, extras: '-e "DOCKER_TAG=${DOCKER_TAG} aksrg=${aksrg} akscls=${akscls}  AZURECLIENTID=${AZURECLIENTID}  AZURECLIENTSECRET=${AZURECLIENTSECRET}  AZURETENANTID=${AZURETENANTID}  AZURESUBSCRIPTIONID=${AZURESUBSCRIPTIONID}"', installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
			  }
            }
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
