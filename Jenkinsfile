stage('CI') {
    node {

        checkout scm
        // git branch: 'jenkins-file',
        //     url: 'https://github.com/bosr/solitaire-systemjs-course'

        // pull dependencies from npm
        // on windows use: bat 'npm install'
        sh 'npm install'

        // stash code & dependencies to expedite subsequent testing
        // and ensure same code & dependencies are used throughout the pipeline
        // stash is a temporary archive
        stash name: 'everything',
              excludes: 'test-results/**',
              includes: '**'
    }
}

stage('browserTests') {
    parallel chrome: {
        runTests('Chrome')
    }, safari: {
        runTests('Safari')
    }, phantom: {
        runTests('PhantomJS')
    }
}

stage('approval') {
    node {
        notify('Deploy to Staging approval required')
    }

    input 'Deploy to Staging?'
}

stage('Deployment to Staging') {
    deploy('staging')
}


def deploy(deployment_target) {
    node {
        sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"

        sh 'docker-compose up -d --build'

        notify("Deployed to ${deployment_target}")
    }
}

def runTests(browser) {
    node {
        sh 'rm -rf *'
        sh 'ls'
        unstash 'everything'
        sh 'ls'
        sh "npm run test-single-run -- --browsers ${browser}"
        // archive karma test results (karma is configured to export junit xml files)
        step([$class: 'JUnitResultArchiver',
              testResults: 'test-results/**/test-results.xml'])
    }
}

def notify(status){
    emailext (
      to: "wesmdemos@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
