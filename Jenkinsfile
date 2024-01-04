node(){
    stage('Build') {
        checkout scm
        sh 'python3 -m py_compile sources/add2vals.py sources/calc.py'
        stash(name: 'compiled-results', includes: 'sources/*.py*')
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
    }
    stage('Manual Approval'){
        input message: 'Apakah hasil sudah OK? (Klik "Proceed" untuk Deploy)'
    }
    stage('Deploy'){
        withEnv(['VOLUME=$(pwd)/sources:/src', 'IMAGE=cdrx/pyinstaller-linux:python3']) {
            dir(path: env.BUILD_ID) {
                unstash name: 'compiled-results'
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                echo 'Kriteria 3, tunggu 1 menit...'
                //sh 'sleep 60'
                archiveArtifacts "sources/dist/add2vals"
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"

                // Upload artifact to GitHub Releases
                script {
                    def githubToken = env.GITHUB_TOKEN  // Set your GitHub token as a secret in Jenkins
                    def artifactPath = "sources/dist/add2vals"
                    def githubRepoUrl = 'https://github.com/Aiki-bot51/simple-python-pyinstaller-app'  // Assume GITHUB_REPO_URL is an environment variable or parameter

                    echo "Constructed URL: ${githubRepoUrl}/releases/latest/assets?name=add2vals"

                    withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                        sh """
                            curl -sSL -H 'Authorization: token \$GITHUB_TOKEN' \
                            -H 'Content-Type: application/octet-stream' \
                            --data-binary @${artifactPath} \
                            '${githubRepoUrl}/releases/latest/assets?name=add2vals'
                        """.stripIndent()
                    }
                }

            }
        }
    }
}