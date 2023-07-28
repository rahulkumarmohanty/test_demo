pipeline {
    agent any
    environment {
        INFRACOST_API_KEY = "ico-v188MHnFUuLsKsSNOYLXfIhnL7ANIkaw"
    }

    parameters {
        choice choices: ['Terraform Init & Plan'], description: 'Choose any one of the Terraform actions to perform..', name: 'terraformaction'
        choice choices: ['VM','Loadbalancer','CDN','Managed disk','Blob storage'], description: 'Choose any one of the Resource to deploy in the Azure Environment..', name: 'resource'
    }

    stages {

        stage('Checkout SCM') {
            steps {
                script {
                    switch(params.resource) {
                        case 'Loadbalancer':
                            checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/rahulkumarmohanty/az_loadbalancer.git']])
                        break
                        case 'CDN':
                            checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_cdn']]) 
                        break
                    }
                }
            }
        }
    }
}