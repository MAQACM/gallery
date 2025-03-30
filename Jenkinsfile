pipeline{
    agent any
    environment{
        GIT_REPO='https://github.com/MAQACM/gallery.git'
        GIT_BRANCH='master'
        PATH = "/usr/local/bin:/opt/homebrew/bin:$PATH"
        RENDER_WEBHOOK="https://api.render.com/deploy/srv-cvk49d9r0fns739mvfe0?key=rPzbeMIChOQ"
        RENDER_SITE='https://gallery-rn2j.onrender.com'
        SLACK_WEBHOOK=credentials('614a3b05-b410-40f4-9ee5-e3dc6fe55d0c')
        HEROKU_URL='https://gallery-mark-ip-1-2f6c401795a5.herokuapp.com'
        HEROKU_APP_NAME='gallery-mark-ip-1'
        HEROKU_API_KEY=credentials("31c31555-3463-46f7-8a4c-b3b32f75696a")
        REPO_OWNER="mark.kasimu@student.moringaschool.com"
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
        stage('Deploy to Heroku') {
                    steps {
                        script {
                            sh """
                                heroku git:remote -a ${HEROKU_APP_NAME}
                                git push https://heroku:$HEROKU_API_KEY@git.heroku.com/${HEROKU_APP_NAME}.git master
                            """
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
                           "text":"*Deployed ${env.REPO_NAME} to both Heroku and Rendor successfully.*\n\nBranch:${GIT_BRANCH}\nRendor DeploymentId:${env.DEPLOYMENT_ID}\nRendor URL:${env.RENDER_SITE}\nHeroku URL:${env.HEROKU_URL}\n"
                        }
                        """.trim()
                        // Send message to Slack
                        sh """
                            curl -X POST -H 'Content-type: application/json' --data '${slackMessage}' ${SLACK_WEBHOOK}
                        """
                    }
            }
            failure {
            //send email to owner incase pipeline fails
                 script {
                                 emailext subject:"JENKINS PIPELINE FAILURE!!!:${REPO_NAME}",
                                          body: """
                                          The Jenkins build for ${env.JOB_NAME} has failed.
                                          \nBranch: ${env.GIT_BRANCH}
                                          \nBuild URL: ${env.BUILD_URL} \n

                                          Please check the logs for more details.
                                          """,
                                          to: "$REPO_OWNER",
                                          from: "jenkins@example.com"
                             }
            }
        }
}