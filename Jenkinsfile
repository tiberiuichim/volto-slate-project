pipeline {
  agent any

  environment {
        GIT_NAME = "volto-slate-project"
        SONARQUBE_TAGS = "www.eionet.europa.eu,forest.eea.europa.eu"
  }

  stages {


    stage('Tests') {
      steps {
        node(label: 'eea') {
            stages {
               stage("install") {
                   steps {
                       script{
                         checkout scm
                         withEnv(["PATH+NODEJS=${tool 'NodeJS12'}/bin"]) { 
                            sh "yarn install"  
                         }
                        }
                   }
               }
               stage("code test") {
                   steps {
                     'Prettier':{
                       script{
                         withEnv(["PATH+NODEJS=${tool 'NodeJS12'}/bin"]) { 
                             sh "yarn run prettier"
                         }
                       }
                     },
                     'Lint':{
                       script{
                         withEnv(["PATH+NODEJS=${tool 'NodeJS12'}/bin"]) { 
                             sh "yarn run code-analysis:i18n"
                         }
                       }
                     }
                     'i18n':{
                       script{
                         withEnv(["PATH+NODEJS=${tool 'NodeJS12'}/bin"]) { 
                             sh "yarn run code-analysis:i18n"
                         }
                       }
                     }
               }
               stage("Unit tests") {
                   steps {
                       script{
                         checkout scm
                         withEnv(["PATH+NODEJS=${tool 'NodeJS12'}/bin"]) { 
                            sh "yarn test-addon --watchAll=false --collectCoverage"  
                         }
                        }
                   }
               }
               stage("Sonarqube") {
                   steps {
                       script{
                         checkout scm
                         withEnv(["PATH+NODEJS=${tool 'NodeJS12'}/bin"]) { 
                             withSonarQubeEnv('Sonarqube') {
                               sh "export PATH=$PATH:${scannerHome}/bin:${nodeJS}/bin; sonar-scanner -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info -Dsonar.sources=./src -Dsonar.projectKey=$GIT_NAME-$BRANCH_NAME -Dsonar.projectVersion=$BRANCH_NAME-$BUILD_NUMBER"
                               sh '''try=2; while [ \$try -gt 0 ]; do curl -s -XPOST -u "${SONAR_AUTH_TOKEN}:" "${SONAR_HOST_URL}api/project_tags/set?project=${GIT_NAME}-${BRANCH_NAME}&tags=${SONARQUBE_TAGS},${BRANCH_NAME}" > set_tags_result; if [ \$(grep -ic error set_tags_result ) -eq 0 ]; then try=0; else cat set_tags_result; echo "... Will retry"; sleep 60; try=\$(( \$try - 1 )); fi; done'''
                             }
                         }
                       }
                   }
               }
            }
          
    stage('Unit Tests') {
      steps {
        node(label: 'eea') {
          script{
            checkout scm
            def scannerHome = tool 'SonarQubeScanner';
            withEnv(["PATH+NODEJS=${tool 'NodeJS12'}/bin"]) {
                sh "env"
                sh "yarn install"
                sh "yarn test-addon --watchAll=false --collectCoverage"
                sh "yarn run prettier"
                sh "yarn run lint"
                sh "yarn run code-analysis:i18n"
                withSonarQubeEnv('Sonarqube') {
                    sh "export PATH=$PATH:${scannerHome}/bin:${nodeJS}/bin; sonar-scanner -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info -Dsonar.sources=./src -Dsonar.projectKey=$GIT_NAME-$BRANCH_NAME -Dsonar.projectVersion=$BRANCH_NAME-$BUILD_NUMBER"
                    sh '''try=2; while [ \$try -gt 0 ]; do curl -s -XPOST -u "${SONAR_AUTH_TOKEN}:" "${SONAR_HOST_URL}api/project_tags/set?project=${GIT_NAME}-${BRANCH_NAME}&tags=${SONARQUBE_TAGS},${BRANCH_NAME}" > set_tags_result; if [ \$(grep -ic error set_tags_result ) -eq 0 ]; then try=0; else cat set_tags_result; echo "... Will retry"; sleep 60; try=\$(( \$try - 1 )); fi; done'''
                }
             }
          }
        }
      }
    }

  }

  post {
    changed {
      script {
        def url = "${env.BUILD_URL}/display/redirect"
        def status = currentBuild.currentResult
        def subject = "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
        def summary = "${subject} (${url})"
        def details = """<h1>${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - ${status}</h1>
                         <p>Check console output at <a href="${url}">${env.JOB_BASE_NAME} - #${env.BUILD_NUMBER}</a></p>
                      """

        def color = '#FFFF00'
        if (status == 'SUCCESS') {
          color = '#00FF00'
        } else if (status == 'FAILURE') {
          color = '#FF0000'
        }

        emailext (subject: '$DEFAULT_SUBJECT', to: '$DEFAULT_RECIPIENTS', body: details)
      }
    }
  }
}
