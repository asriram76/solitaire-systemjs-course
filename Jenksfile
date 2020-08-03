stage 'CI'
node {

    env.NODEJS_HOME = "${tool 'node14'}"
    env.PATH="${env.NODEJS_HOME}/bin:${env.PATH}"
    sh "node -v"

    checkout scm
    //git branch: 'jenkins2-course', 
        url: 'https://github.com/g0t4/solitaire-systemjs-course'

    // pull dependencies from npm
    // on windows use: bat 'npm install'
    
    //sh "${nodeHome}/bin/npm install"
    sh "npm install"
    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'
    
    // test with PhantomJS for "fast" "generic" results
    // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
    sh "npm run test-single-run -- --browsers PhantomJS"
    
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: 'JUnitResultArchiver', 
          testResults: 'test-results/**/test-results.xml'])
          
}

node('mac'){
    sh 'ls'
    sh 'rm -rf *'
    unstash 'everything'
    sh 'ls'
}

stage 'Browser testing'

parallel chrome:{
    runTests("Chrome")
}


node {
    notify("Deploy to staging?")
}

input 'Deploy to staging?'

stage name: 'Deploy', concurrency: 1
node{
    sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
    sh 'docker-compose up -d --build'
    notify 'Solitaire deployed!'
}

def runTests(browser){
    node {
        sh 'rm -rf *'
        unstash 'everything'
        sh "npm run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver', 
          testResults: 'test-results/**/test-results.xml'])
    }
}
def notify(status){
    emailext (
      to: "asriram76@yahoo.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
