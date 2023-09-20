pipeline {
  agent any
  environment {
    SOURCE_DIR        = "/var/lib/jenkins/workspace/HRM_V1"
    DEST_DIR          = "/home/sadmin/HRM_V1/backups"
    TIMESTAMP         = sh(returnStdout: true, script: 'date "+%Y-%m-%d_%H-%M-%S"').trim()
    JOB_TIME          = sh(returnStdout: true, script: 'date "+%Y-%m-%d __ %H:%M:%S"').trim()
    ZIP_FILENAME      = "HRMS_V1_${TIMESTAMP}.zip"
    GIT_API_PAT_TOKEN = credentials('Privatepr_1')
    EMAIL_TO_PEOPLE   = "vigneshwaran.sivasubramanian@quadgen.com"
    
  }

  
  stages {

    stage('Capture PR Details') {
            steps {
                script {
                    def gitCommit = sh(script: 'git log --format=%H -n 1', returnStdout: true).trim()

                    def prDetailsJson = sh(script: """
                        curl -L -H "Accept: application/vnd.github+json" \
                        -H "Authorization: Bearer $GIT_API_PAT_TOKEN" \
                        -H "X-GitHub-Api-Version: 2022-11-28" \
                        https://api.github.com/repos/vignesh-s19/HRM_1/commits/$gitCommit/pulls
                    """, returnStdout: true).trim()

                    // Parse the JSON response using JsonSlurper
                    def jsonSlurper = new groovy.json.JsonSlurper()
                    def prDetails = jsonSlurper.parseText(prDetailsJson)

                    // Extract title, body, and ID
                    def prTitle  = prDetails.title
                    def prBody   = prDetails.body
                    def prNUM    = prDetails.number

                    // Set environment variables
                    env.PR_TITLE = prTitle
                    env.PR_BODY = prBody
                    env.PR_NUM = prNUM

                    //Print the PR Details
                    echo "Pull Request Title: ${env.PULL_REQUEST_TITLE}"
                    echo "Pull Request Body: ${env.PULL_REQUEST_BODY}"
                    echo "Pull Request NUM : ${env.PULL_REQUEST_NUM}"
                }
            }
        }
		 stage('Conditional Stage') {
            steps {
                script {
                    def output
                    if (env.PR_BODY == "[null]") {
                      output = sh(script: 'echo "Not available"', returnStdout: true).trim()
                      echo "The output is: ${output}"
                      env.PR_BODY_OUTPUT = output
                      echo "Vikki: ${PR_BODY_OUTPUT}"
                        return;
                    } else {
                      output = sh(script: '"${env.PR_BODY}"', returnStdout: true).trim()

                    }
                }
            }
        }

      stage('E-Mail Notification') {
        steps {
           script {
                     // Defining the email subject along with PR Details
                    def emailSubject = "On-Premise: CICD Pipeline #${env.BUILD_NUMBER}  ${env.JOB_NAME} is Triggered for PR: #${env.PR_NUM}  Title:${env.PR_TITLE}"

                    // Defining the email body contents along with PR Details
                    def emailBody = """
                        Please find the application details for the Pipeline triggered,

                        The HRM PROD pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER} is just started and initiated deployment at [ ${JOB_TIME} ] on ${env.NODE_NAME} Jenkins NODE.


                        Pull Request details:

                        Pull Request Title  : ${env.PR_TITLE}
                        Pull Request Body   : ${env.PR_BODY}
                        Pull Request Number : ${env.PR_NUM}
                        
                        
                        """

              emailext (
                   attachLog: true,
                   subject: emailSubject,
                   body: emailBody,
                   to: "${EMAIL_TO_PEOPLE}",
                   replyTo: "${EMAIL_REPLY_TO}"
                  )
                }
            }
        }

    
    stage('HRM_Backup') {
      steps {
        script {
          echo 'Initiated and started creating the Zip Archive' 
          sh "zip -r ${DEST_DIR}/${ZIP_FILENAME} ${SOURCE_DIR}"
          sh 'pwd'
          sh 'ls -lla'
        }
      }
    }
    stage('HRM_CleanUP') {
      steps {
        script {
          echo 'Initiated and started wiping the existing infrastructures' 
          sh 'pwd'
          sh 'ls -lla'
          sh 'whoami'
          sh 'docker images && docker ps -a'
          // sh 'docker rm -f hrms_v2.dotnet.api.container'
          // sh 'docker rm -f hrms_v2.angular.frontend.container'
          // sh 'docker rmi hrms_v2.dotnet.api.image'
          // sh 'docker rmi hrms_v2.angular.frontend.image'
          // sh 'docker images && docker ps -a'

        }
      }
    }
    stage('HRM_Deploy') {
      steps {
        script {
          echo 'Initiated and started the new infrastructure deployment' 
          sh 'pwd'
          sh 'ls -lla'
        //  // sh "docker-compose build --no-cache"
          sh "docker-compose up --build -d"
        }
      }
    }
  }
  post {
        success {
          cleanWs()
        script {            

                    // Defining the email subject along with PR Details
                    def emailSubject = "On-Premise: CICD Pipeline #${env.BUILD_NUMBER}  ${env.JOB_NAME} is Success for PR: #${env.PR_NUM}  Title:${env.PR_TITLE}"

                    // Defining the email body contents along with PR Details
                    def emailBody = """
                        Please find the application details for the Pipeline,

                        The HRM PROD pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER} is successfully deployed on ${env.NODE_NAME} Jenkins NODE.


                        Application Name : Quadgen HRM Portal
                        Application URL  : http://10.2.126.234

                        Pull Request details:

                        Pull Request Title  : ${env.PR_TITLE}
                        Pull Request Body   : ${env.PR_BODY}
                        Pull Request Number : ${env.PR_NUM}
                        

                        Pipline JOB URL: $BUILD_URL  //Can be accessed by only DevOps Team
                        
                        """

              emailext (
                   attachLog: true,
                   subject: emailSubject,
                   body: emailBody,
                   to: "${EMAIL_TO_PEOPLE}",
                   replyTo: "${EMAIL_REPLY_TO}"
                  )

        }
        }
        failure {
          cleanWs()
        script {            

                    // Defining the email subject along with PR Details
                    def emailSubject = "On-Premise: CICD Pipeline #${env.BUILD_NUMBER}  ${env.JOB_NAME} is Failed for PR: #${env.PR_NUM}  Title:${env.PR_TITLE}"

                    // Defining the email body contents along with PR Details
                    def emailBody = """
                        Please find the application details for the Pipeline,

                        The HRM PROD pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER} is failed on ${env.NODE_NAME} Jenkins NODE.


                        Application Name : Quadgen HRM Portal
                        Application URL  : http://10.2.126.234

                        Pull Request details:

                        Pull Request Title  : ${env.PR_TITLE}
                        Pull Request Body   : ${env.PR_BODY}
                        Pull Request Number : ${env.PR_NUM}
                        

                        Pipline JOB URL: $BUILD_URL  //Can be accessed by only DevOps Team
                        
                        """

              emailext (
                   attachLog: true,
                   subject: emailSubject,
                   body: emailBody,
                   to: "${EMAIL_TO_PEOPLE}",
                   replyTo: "${EMAIL_REPLY_TO}"
                  )
                  
        }
    }
    }
}
