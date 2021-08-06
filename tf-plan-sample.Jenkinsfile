
    pipeline {
        agent any
        triggers { pollSCM('* * * * *') }
        stages {
            stage('Setup') {
                steps {
                        script {
                            dir("workdir") {
                                git url: "https://github.com/RiflerRick/nginx-terraform.git", branch: "main"
                            }
                            
                            sh '''
                            #!/bin/bash
                            curl -u riflerrick:$GITHUB_PAT  -X POST -H "Accept: application/vnd.github.v3+json"  https://api.github.com/repos/riflerrick/nginx-terraform/statuses/$GIT_COMMIT -d '{"state":"pending","target_url":"$BUILD_URL","context":"jenkins-tf-validation"}'
                            '''

                            dir("workdir") {
                                sh ''''
                                #!/bin/bash
                                terraform init
                                '''
                            }
                        }
                }
            }
            stage('Plan Confirmation') {
                
                steps {
                    script {
                        dir("workdir") {
                            sh 'terraform plan'
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
                    sh '''
                    #!/bin/bash
                    curl -u riflerrick:$GITHUB_PAT  -X POST -H "Accept: application/vnd.github.v3+json"  https://api.github.com/repos/riflerrick/nginx-terraform/statuses/$GIT_COMMIT -d '{"state":"success","target_url":"$BUILD_URL","context":"jenkins-tf-validation"}'
                    '''
                }
            }
        }
    }
