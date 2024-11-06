pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }

    environment {
        GIT_CREDENTIALS_ID = 'ms-bit'
        SERVER_HOST = credentials('prj_ip')
        SSH_CREDENTIALS = 'prj-ssh'
    }

    stages {
        stage('Clone repository') { 
            steps { 
                script {
                    checkout scm
                }
            }
        }

        stage('Prepare Environment on Stage') {
            when {
                branch 'stage'
            }
            steps {
                script {
                    sshagent(credentials: [SSH_CREDENTIALS]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_HOST} << EOF
                        chmod -R 777 /var/www/prj/server-scripts
                        cd /var/www/prj/server-scripts
                        bash ./stage_deploy_cypress_rollback_prj.sh
                        exit
                        EOF
                        """
                    }
                }
            }
        }
        // test trigger stage cypress
        // stage('Manual Approval') {
        //     when {
        //         branch 'master'
        //     }
        //     steps {
        //         input message: 'Approve deployment to Production?', ok: 'Deploy'
        //     }
        // }

        // stage('Prepare Environment on Production') {
        //     when {
        //         branch 'master'
        //     }
        //     steps {
        //         script {
        //             sshagent(credentials: [SSH_CREDENTIALS]) {
        //                 sh """
        //                 ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_HOST} << EOF
        //                 chmod -R 777 /var/www/prj/server-scripts
        //                 cd /var/www/prj/server-scripts
        //                 bash ./master_deploy_cypress_rollback_prj.sh
        //                 exit
        //                 EOF
        //                 """
        //             }
        //         }
        //     }
        // }       
        
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
            cleanWs() // Cleans workspace after pipeline finishes
            emailext to: "ahamedsubhani12@gmail.com",
                subject: "jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}",
                body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}\nMore Info can be found here: ${env.BUILD_URL}"
        }
    }
}
