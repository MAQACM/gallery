pipeline{
    agent any
    environment{
        GIT_REPO='https://github.com/MAQACM/gallery.git'
        GIT_BRANCH='master'
        PATH = "/usr/local/bin:/opt/homebrew/bin:$PATH"
        RENDER_WEBHOOK="https://api.render.com/deploy/srv-cvk49d9r0fns739mvfe0?key=rPzbeMIChOQ"
        RENDER_SITE='https://gallery-rn2j.onrender.com'
        SLACK_WEBHOOK='https://hooks.slack.com/services/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'
    }
    stages{
        stage("clone code"){
            steps{
                git  branch:"${GIT_BRANCH}", url:"${GIT_REPO}"
            }
        }
        stage('Install Dependencies') {
                    steps {
                        sh 'npm install'
                    }
                }
        stage("test code"){
            steps{
                sh "npm test"
            }
        }
        stage("build app"){
            steps{
                sh 'npm install'
            }
        }
        stage("deploy app to render"){
            steps{
                script {
                    env.RENDER_RESP=sh(script:"""
                        curl -X POST "${RENDER_WEBHOOK}" \
                        -H "Content-Type: application/json"
                    """,returnStdout:true).trim()
                    echo "Render deployment response:${env.RENDER_RESP}"
                }
            }
        }

    }
      post {
            always {
                script {
                    echo " Pipeline Execution Completed!"
                    env.REPO_NAME = sh(script: "basename -s .git `git config --get remote.origin.url`", returnStdout: true).trim()
                    env.DEPLOYMENT_ID =sh(script: "echo '${env.RENDER_RESP}' | jq -r '.deploy.id'", returnStdout: true).trim()
                }
            }
            success {
                 script{
                        // Prepare Slack message
                        def slackMessage = """
                        {
                           "text":"*Render deployed ${env.REPO_NAME} successfully.*\n\nBranch:${GIT_BRANCH}\nDeploymentId:${env.DEPLOYMENT_ID}\nDeployment URL:${env.RENDER_SITE}\n"
                        }
                        """.trim()
                        // Send message to Slack
                        sh """
                            curl -X POST -H 'Content-type: application/json' --data '${slackMessage}' ${SLACK_WEBHOOK}
                        """
                    }
            }
            failure {
                 script{
                        // Prepare Slack message
                        def slackMessage = """
                        {
                            "text": "*Deployemnt for ${env.REPO_NAME} failed kindy check jenkins console for detailed logs.*\nBranch:${GIT_BRANCH}\n"
                        }
                        """.trim()
                        // Send message to Slack
                        sh """
                            curl -X POST -H 'Content-type: application/json' --data '${slackMessage}' ${SLACK_WEBHOOK}
                        """
                    }
            }
        }
}