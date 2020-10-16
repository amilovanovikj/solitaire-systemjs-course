pipeline {
    agent any
    stages {
        // continuous integration
        stage('Notify') {
            steps {
                notify('Build started')
            }
        }
        stage('Checkout SCM'){
            steps {
                checkout scm
            }
            // git branch: 'jenkins2-course', 
            //     url: 'https://github.com/g0t4/solitaire-systemjs-course'
        }
        stage('Pull dependencies'){
            steps {
                sh 'npm install'
            }
        }
        stage('Stash code & dependencies'){
            steps {
                stash name: 'everything', 
                    excludes: 'test-results/**', 
                    includes: '**'
            }
        }
        stage('Test'){
            steps {
                sh 'npm run test-single-run -- --browsers PhantomJS'
            }
        }
        stage('Archive'){
            steps {
                step([$class: 'JUnitResultArchiver', 
                    testResults: 'test-results/**/test-results.xml'])
                archiveArtifacts artifacts: 'app/', followSymlinks: false
            }
        }

        // demoing a second agent
        stage('Clean up'){
            steps {
                node('ubuntu'){
                    sh 'ls'
                    sh 'rm -rf *'
                    unstash 'everything'
                    sh 'ls'
                }
            }
        }

        // parallel integration testing
        stage('Browser testing'){
            steps {
                script {
                    try {
                        parallel chrome: {
                            runTests("Chrome")
                        }, firefox: {
                            runTests("Firefox")
                        }, safari: {
                            runTests("Safari")
                        }
                    } catch(err) {
                        notify("Error: ${err}")
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }

        // deployment
        stage('Approve deployment') {
            steps {
                node('master') {
                    notify("Deploy to staging?")
                }
                input 'Deploy to staging?'
            }
        }

        stage('Deploy') {
            steps{
                node('master') {
                    sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
                    sh 'docker-compose up -d --build'
                    notify('App Deployed! You can find it on localhost:3000')
                }
            }
        }
    }
}

def runTests(browser) {
    node {
        sh 'rm -rf *'
        unstash 'everything'
        sh "npm run test-single-run -- -browsers ${browser}"
        step([$class: 'JUnitResultArchiver', 
            testResults: 'test-results/**/test-results.xml'])
    }
}

def notify(status){
    emailext (
    to: "me@gmail.com",
    subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
    body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}