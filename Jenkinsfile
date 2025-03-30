pipeline{
    agent any
    environment{
        GIT_REPO='https://github.com/MAQACM/gallery.git'
        GIT_BRANCH='master'
        PATH = "/usr/local/bin:/opt/homebrew/bin:$PATH"
        RENDER_WEBHOOK="https://api.render.com/deploy/srv-cvk49d9r0fns739mvfe0?key=rPzbeMIChOQ"
    }
    stages{
        stage("clone code"){
            steps{
                git  branch:"${GIT_BRANCH}", url:"${GIT_REPO}"

            }
        }
        stage("install dependancies"){
            steps{
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
                sh 'npm run build'
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
}