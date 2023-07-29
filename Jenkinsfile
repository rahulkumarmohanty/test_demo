pipeline {
    agent any
    environment {
        INFRACOST_API_KEY = "ico-v188MHnFUuLsKsSNOYLXfIhnL7ANIkaw"
    }

    parameters {
        choice choices: ['Terraform Init & Plan'], description: 'Choose any one of the Terraform actions to perform..', name: 'terraformaction'
        choice choices: ['VM','Loadbalancer','CDN','Managed disk','Blob storage'], description: 'Choose any one of the Resource to deploy in the Azure Environment..', name: 'resource'
        string(name: 'ARM_TENANT_ID', description: 'Enter the tenant id')
        string(name: 'ARM_CLIENT_ID', description: 'Enter the client id')
        string(name: 'ARM_CLIENT_SECRET', description: 'Enter the client secret')
        string(name: 'ARM_SUBSCRIPTION_ID',description: 'Enter the subscription id')
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
                    archiveArtifacts artifacts:'*', onlyIfSuccessful: true
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
