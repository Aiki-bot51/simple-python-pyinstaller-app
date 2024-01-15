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
//    stage('Manual Approval'){
//        input message: 'Apakah hasil sudah OK? (Klik "Proceed" untuk Deploy)'
//    }
    stage('Deploy') {
        withCredentials([string(credentialsId: 'vercel-credentials', variable: 'VERCEL_TOKEN')]) {
            withEnv(['VOLUME=$(pwd)/sources:/src', 'IMAGE=cdrx/pyinstaller-linux:python3']) {
                dir(path: env.BUILD_ID) {
                    unstash name: 'compiled-results'
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                    echo 'Kriteria 3, tunggu 1 menit...'
                    archiveArtifacts "sources/dist/add2vals"

                    echo 'Debug information:'
                    sh "ls -la /var/jenkins_home/workspace/submission-cicd-pipeline-aikyodzakia/181/sources/dist"

                    dir("sources/dist") {
                        sh "ls -la"

                        // Check if the project exists on Vercel
                        def projectExists = sh(script: "vercel --token=\${VERCEL_TOKEN} inspect \$(pwd)/sources/dist", returnStatus: true)

                        // Deploy or redeploy based on project existence
                        if (projectExists == 0) {
                            echo "Vercel project exists. Triggering redeploy..."
                            sh "vercel --token=\${VERCEL_TOKEN} --prod --confirm \$(pwd)/sources/dist -y"
                        } else {
                            echo "Vercel project does not exist. Deploying for the first time..."
                            sh "vercel --token=\${VERCEL_TOKEN} --prod \$(pwd)/sources/dist -y"
                        }
                        
                        // Rename the deployed file to have a .py extension
                        sh "mv add2vals add2vals.py"
                    }
                }
            }
        }
    }
}