pipeline {
  agent any
  environment {

    TIMESTAMP         = sh(returnStdout: true, script: 'date "+%Y-%m-%d_%H-%M-%S"').trim()
    JOB_TIME          = sh(returnStdout: true, script: 'date "+%Y-%m-%d __ %H:%M:%S"').trim()
    ZIP_FILENAME      = "HRMS_V1_${TIMESTAMP}.zip"
    GIT_API_PAT_TOKEN = credentials('Pr_token')
    EMAIL_PEOPLE   = "vigneshwaran.sivasubramanian@quadgen.com"
    
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

		    // Remove special characters from title and description
                    prTitle = prTitle.replaceAll(/[^a-zA-Z0-9\s]/, '')
                    prBody = prBody.replaceAll(/[^a-zA-Z0-9\s]/, '')

                    // Set environment variables
                    env.PR_TITLE = prTitle
                    env.PR_BODY = prBody
                    env.PR_NUM = prNUM

		    echo "The value is: ${PR_BODY}"
		    echo "The value is: ${PR_TITLE}"
                }
            }
        }
		 // stage('Conditional Stage') {
   //          steps {
   //              script {
   //                  def output
   //                  if (env.PR_BODY == "[null]") {
   //                    output = sh(script: 'echo "Not available"', returnStdout: true).trim()
   //                    echo "The output is: ${output}"
   //                    env.PR_BODY_OUTPUT = output
   //                    echo "Vikki: ${PR_BODY_OUTPUT}"
   //                      return;
   //                  } else {
   //                    output = sh(script: '"${env.PR_BODY}"', returnStdout: true).trim()

   //                  }
   //              }
   //          }
   //      }

                    // Switch statement for validation
	  stage('switch condition') {
		  steps{
			  script{
                    		switch (prBody) {
                     		case 'null':
                          	echo "Description matches 'null'"
                        	break
                        default:
                            echo "Description has some expected value"
                       }
		    }
	        }
             }
  
}
}
