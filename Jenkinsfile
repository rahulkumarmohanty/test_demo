pipeline {
    agent any
    environment {
        INFRACOST_API_KEY = "ico-v188MHnFUuLsKsSNOYLXfIhnL7ANIkaw"
    }

    parameters {
        choice choices: ['Terraform Init & Plan'], description: 'Choose any one of the Terraform actions to perform..', name: 'terraformaction'
        choice choices: ['VM','Loadbalancer','CDN','ManagedDisk','Application gateway','Blob storage','nsg','postgres','postgres_cosmosdb','snapshot','vpn_vnet_gw','image'], description: 'Choose any one of the Resource to deploy in the Azure Environment..', name: 'resource'
        string(name: 'ARM_TENANT_ID', defaultValue: '73306657-9352-491f-8fcc-40bf911a22e9', description: 'Enter the tenant id')
        string(name: 'ARM_CLIENT_ID', defaultValue: '8599eb49-abbc-416f-aeed-1c33b31fe714', description: 'Enter the client id')
        string(name: 'ARM_CLIENT_SECRET', defaultValue: 'yrr8Q~B0yFyo3ZG58dDK87Jgx1VEEZUadi_1bcdG', description: 'Enter the client secret')
        string(name: 'ARM_SUBSCRIPTION_ID', defaultValue: 'd4d1547b-1e0c-4d70-9407-79b556cb3687', description: 'Enter the subscription id')
        string(name: 'tfvars',description: 'Enter the tfvars file name')
        string(name: 'account',description: 'Enter the account name')
    }

    stages {

        stage('Checkout SCM') {
            steps {
                script {
                    switch(params.resource) {
                        case 'VM':
                            cleanWs()
                            checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_vm']])
                        break
                        case 'CDN':
                            cleanWs()
                            checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_cdn']]) 
                        break
                        case 'Blob storage':
                            cleanWs()
                            checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_blob_storage']])
                        break
                        case 'Application gateway':
                            cleanWs()
                            checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_app_gateway']])
                        break
                        case 'Loadbalancer':
                            cleanWs()
                            checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_loadbalancer']])
                        break
                        case 'ManagedDisk':
                            cleanWs()
                            checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_managed_disk']])
                        break
                        case 'nsg':
                            cleanWs()
                            checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_nsg']])
                        break
                        case 'postgres':
                            cleanWs()
                            checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_postgres']])
                        break
                        case 'postgres_cosmosdb':
                            cleanWs()
                            checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_postgres_cosmosdb']])
                        break
                        case 'snapshot':
                            cleanWs()
                            checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_snapshot']])
                        break
                        case 'vpn_vnet_gw':
                            cleanWs()
                            checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_vpn_vnet_gw']])
                            def inputFile = input message: 'Upload file', parameters: [file(name: 'cert')]
                            new hudson.FilePath(new File("$workspace/cert")).copyFrom(inputFile)
                            inputFile.delete()
                        break
                        case 'image':
                            cleanWs()
                            checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'SanthaID', url: 'https://git-codecommit.ap-south-1.amazonaws.com/v1/repos/az_image']])
                        break
                    }
                }
            }
        }

        stage('azure cli logging') {
            steps{
                sh 'az login --service-principal --username ${ARM_CLIENT_ID} --password ${ARM_CLIENT_SECRET} --tenant ${ARM_TENANT_ID}'
                sh 'az account set --subscription ${ARM_SUBSCRIPTION_ID}'
            }
        }

        stage('terraform state creation') {
            steps{
                script{
                    def content = "key = \"${account}/${resource}/tfstate${env.BUILD_NUMBER}\""
                    writeFile file: 'backend-conffinal.tfvars', text: content
                }
            }
        }

        stage('Terraform Initialize') {
            steps {
                sh 'terraform init --backend-config=backend-conffinal.tfvars -reconfigure'
            }
        }

        stage('TFLint') {
            steps {
                sh 'tflint --ignore-module=SweetOps/storage-bucket/aws'
            }
        }

        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    def tfplan = sh(script: 'terraform plan --var-file=${tfvars} -out=myplan.tfplan -input=false', returnStdout: true).trim()
                    writeFile file: 'tfplan', text: tfplan
                }
            }
            post {
                always{
                    archiveArtifacts artifacts:'**/*.*', onlyIfSuccessful: true
                }
            }
        }

        stage('Terraform Plan View') {
            steps {
                sh 'terraform show myplan.tfplan > readable_plan.txt'
                sh 'cat readable_plan.txt'
            }
        }

        stage('Tfsec Check for potential security issues') {
            steps {
                script {
                  try {
                    sh 'tfsec .'
                } catch (Exception e) {
                    // handle the error or ignore it
                        echo "Error occurred: ${e.message}"
                }
             }
          }
        }

        stage('Checkov') {
            steps {
                script {
                  try {
                    sh 'checkov -d .'
                } catch (err) {
                  echo 'Checkov check failed, continuing pipeline'
                  echo err.getMessage()
                }
              }
           }
        }

        stage('Infracost Breakdown') {
            steps {
                sh 'infracost breakdown --path myplan.tfplan'
            }
        }

        stage('Drift Detection') {
            steps {
                script {
                  try {
                    sh 'driftctl scan --from tfstate+s3://1ch-aws-terraform/terraform.tfstate'
                } catch (Exception e) {
                    // handle the error
                    echo "Error occurred: ${e.message}"
                }
                build job: 'test2', wait: false, parameters: [string(name: 'ARM_TENANT_ID', value: String.valueOf(ARM_TENANT_ID)), string(name: 'ARM_CLIENT_ID', value: String.valueOf(ARM_CLIENT_ID)), string(name: 'ARM_CLIENT_SECRET', value: String.valueOf(ARM_CLIENT_SECRET)), string(name: 'ARM_SUBSCRIPTION_ID', value: String.valueOf(ARM_SUBSCRIPTION_ID))]
             }
          }
        }
    }
}
