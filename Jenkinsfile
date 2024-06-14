pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }

    environment {
        GIT_CREDENTIALS_ID = 'ms-bit'
        UAT_SERVER = 'prj'
        PROD_SERVER = 'prj'
        DEPLOY_PATH_UAT = 'wp-content/themes-plugins/'
        DEPLOY_PATH_PROD = 'wp-content/themes-plugins/'
        SOURCE_FILE_1 = 'wp-content/plugins/'
        SOURCE_FILE_2 = 'wp-content/themes/'
        PREFIX = 'wp-content'
        GIT_USERNAME = credentials("GIT_USERNAME")
        GIT_PASSWORD = credentials("GIT_PASSWORD")
    }

    stages {
        stage('Clone repository') { 
            steps { 
                script {
                    checkout scm
                }
            }
        }

        stage('Prepare Environment on UAT') {
            when {
                branch 'feature/jenkins-multibranch'
            }
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: env.UAT_SERVER,
                            transfers: [
                                sshTransfer(
                                    sourceFiles: env.SOURCE_FILE_1,
                                    remoteDirectory: env.DEPLOY_PATH_UAT,
                                    removePrefix: env.PREFIX,
                                    execCommand: 'echo "deploy plugins on wp-content for prj dev"'
                                ),
                                sshTransfer(
                                    sourceFiles: env.SOURCE_FILE_2,
                                    remoteDirectory: env.DEPLOY_PATH_UAT,
                                    removePrefix: env.PREFIX,
                                    execCommand: 'echo "deploy themes on wp-content for prj dev"'
                                )
                            ],
                            usePromotionTimestamp: false,
                            useWorkspaceInPromotion: false,
                            verbose: true,                  
                        )
                    ]
                )
                sshagent(credentials: ['prj-ssh']) {
                    script {
                        sh '''
                            ssh -o StrictHostKeyChecking=no username@ip << EOF
                            chmod -R 777 /var/www/prj-bkp/server-scripts
                            cd /var/www/prj-bkp/server-scripts
                            sh ./delete_move_themes_plugins_stage.sh
                            exit
                            EOF
                        '''
                    }
                }
            }
        }

        stage('Prepare Environment on Stage') {
            when {
                branch 'stage'
            }
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: env.UAT_SERVER, // Assuming stage server is same as UAT
                            transfers: [
                                sshTransfer(
                                    sourceFiles: env.SOURCE_FILE_1,
                                    remoteDirectory: env.DEPLOY_PATH_UAT,
                                    removePrefix: env.PREFIX,
                                    execCommand: 'echo "deploy plugins on wp-content for prj dev"'
                                ),
                                sshTransfer(
                                    sourceFiles: env.SOURCE_FILE_2,
                                    remoteDirectory: env.DEPLOY_PATH_UAT,
                                    removePrefix: env.PREFIX,
                                    execCommand: 'echo "deploy themes on wp-content for prj dev"'
                                )
                            ],
                            usePromotionTimestamp: false,
                            useWorkspaceInPromotion: false,
                            verbose: true,
                        )
                    ]
                )
                sshagent(credentials: ['prj-ssh']) {
                    script {
                        sh '''
                            ssh -o StrictHostKeyChecking=no username@ip << EOF
                            chmod -R 777 /var/www/prj-bkp/server-scripts
                            cd /var/www/prj-bkp/server-scripts
                            sh ./delete_move_themes_plugins_stage.sh
                            exit
                            EOF
                        '''
                    }
                }
            }
        }

        stage('Manual Approval') {
            when {
                branch 'master'
            }
            steps {
                input message: 'Approve deployment to Production?', ok: 'Deploy'
            }
        }

        stage('Prepare Environment on Production') {
            when {
                branch 'master'
            }
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: env.PROD_SERVER,
                            transfers: [
                                sshTransfer(
                                    sourceFiles: env.SOURCE_FILE_1,
                                    remoteDirectory: env.DEPLOY_PATH_PROD,
                                    removePrefix: env.PREFIX,
                                    execCommand: 'echo "deploy plugins on wp-content for prj prod"'
                                ),
                                sshTransfer(
                                    sourceFiles: env.SOURCE_FILE_2,
                                    remoteDirectory: env.DEPLOY_PATH_PROD,
                                    removePrefix: env.PREFIX,
                                    execCommand: 'echo "deploy themes on wp-content for prj prod"'
                                )
                            ],
                            usePromotionTimestamp: false,
                            useWorkspaceInPromotion: false,
                            verbose: true,
                        )
                    ]
                )
                sshagent(credentials: ['prj-ssh']) {
                    script {
                        sh '''
                            ssh -o StrictHostKeyChecking=no username@ip << EOF
                            chmod -R 777 /var/www/prj-bkp/server-scripts
                            cd /var/www/prj-bkp/server-scripts
                            sh ./delete_move_themes_plugins_stage.sh
                            exit
                            EOF
                        '''
                    }
                }
            }
        }

        stage('Rollback to Previous Version') {
            when {
                anyOf {
                    branch 'feature/jenkins-multibranch'
                    branch 'stage'
                    branch 'master'
                }
            }
            steps {
                input message: 'Rollback to previous version?', ok: 'Rollback'
                script {
                    withCredentials([usernamePassword(credentialsId: env.GIT_CREDENTIALS_ID, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh """
                        cd /var/www/prj-bkp/
                        git checkout HEAD~1
                        // git add .
                        // git commit -m "Rollback to previous version"
                        // git push https://$GIT_USERNAME:$GIT_PASSWORD@bitbucket.org/pixacore/prj.git HEAD:master
                        """
                    }
                }
            }
        }

        stage('Checkout code') {
            steps {
                script {
                    def scmVars = checkout([
                        $class: 'GitSCM',
                    ])

                    echo "scmVars.GIT_COMMIT"
                    echo "${scmVars.GIT_COMMIT}"

                    env.GIT_COMMIT = scmVars.GIT_COMMIT
                    echo "env.GIT_COMMIT"
                    echo "${env.GIT_COMMIT}"
                }
            }
        }
    }

    post {
        always {
            emailext to: "ahamedsubhani12@gmail.com",
                subject: "jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}",
                body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}\nMore Info can be found here: ${env.BUILD_URL}"
        }
    }
}