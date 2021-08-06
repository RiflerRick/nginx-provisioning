
    pipeline {
        agent any
        stages {
            stage('Checkout') {
                steps {
                    dir("workdir") {
                        checkout scm: [$class: 'GitSCM',
                            userRemoteConfigs: [[url: 'https://github.com/RiflerRick/nginx-terraform.git']],
                                                branches: [[name: 'refs/heads/main']]
                            ], poll: true, changelog: true
                    }
                    
                }
            }
            stage('Setup') {
                steps {
                        script {
                            withCredentials([string(credentialsId: 'GITHUB_PAT', variable: 'GITHUB_PAT')]) {
                                dir("workdir") {
                                    sh '''
                                    #!/bin/bash
                                    current_commit=$(git log --name-status HEAD^..HEAD | grep commit | cut -d " " -f 2)
                                    echo $BUILD_URL
                                    curl -u riflerrick:$GITHUB_PAT  -X POST -H "Accept: application/vnd.github.v3+json"  https://api.github.com/repos/riflerrick/nginx-terraform/statuses/$current_commit -d '{"state":"pending","target_url":"https://jenkins-lz.globalpay.com/","context":"jenkins-tf-validation"}'
                                    '''
                                }
                                
                            }
                            

                            dir("workdir") {
                                withCredentials([file(credentialsId: 'global-image-sharing-svc-account-key', variable: 'svc_account_key')]) {
                                    writeFile file: 'svc-account-key.json', text: readFile(svc_account_key)
                                    sh '''
                                    #!/bin/bash
                                    terraform init
                                    '''
                                }
                            }
                        }
                }
            }
            stage('Plan Confirmation') {
                
                steps {
                    script {
                        dir("workdir") {
                            withCredentials([file(credentialsId: 'global-image-sharing-svc-account-key', variable: 'svc_account_key')]) {
                                writeFile file: 'svc-account-key.json', text: readFile(svc_account_key)
                                sh '''
                                #!/bin/bash
                                terraform plan
                                '''
                            }
                        }
                        

                        timeout(time: 1, unit: 'HOURS') {
                            input(id: 'confirm', message: 'Apply Terraform?')
                        }
                    }
                }
            }
            stage('Terraform Apply') {
                
                steps {
                    dir('workdir') {
                        sh 'echo "will apply"'
                    }
                }
            }
            stage('Post Apply') {
                steps {
                    withCredentials([string(credentialsId: 'GITHUB_PAT', variable: 'GITHUB_PAT')]) {
                        dir("workdir") {
                            sh '''
                            #!/bin/bash
                            current_commit=$(git log --name-status HEAD^..HEAD | grep commit | cut -d " " -f 2)
                            echo $BUILD_URL
                            curl -u riflerrick:$GITHUB_PAT  -X POST -H "Accept: application/vnd.github.v3+json"  https://api.github.com/repos/riflerrick/nginx-terraform/statuses/$current_commit -d '{"state":"success","target_url":"https://jenkins-lz.globalpay.com/","context":"jenkins-tf-validation"}'
                            '''
                        }
                    }
                }
            }
        }
    }
