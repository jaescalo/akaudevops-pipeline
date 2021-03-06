pipeline {
  agent any

  environment {
      PIPELINE_ENV = 'dev'
  }

  stages {
    stage ("GitHub Checkout dev") {
      steps {
        echo "Step 1: Checkout Git dev Branch"
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/dev']],
          doGenerateSubmoduleConfigurations: false,
          extensions: [
            [$class: 'RelativeTargetDirectory', relativeTargetDir: 'jaescalo.edge.akau.webperf.it']],
          submoduleCfg: [],
          userRemoteConfigs: [
            [credentialsId: '5153d0dc-5894-4cbe-9790-bdb840ec1734', url: 'https://github.com/jaescalo/akaudevops-pipeline.git', name: 'akaudevops-pipeline']]
        ])
      }
    }
    stage ("Save Changes to dev Env") {
      steps {
        echo "Step 2: Save Changes to dev Env. Job: Jenkins_${JOB_NAME}-${BUILD_NUMBER}"
        sh "source ~/.bash_profile 2> /dev/null; akamai pipeline save -p jaescalo.edge.akau.webperf.it ${PIPELINE_ENV}"
        sh "source ~/.bash_profile 2> /dev/null; akamai pipeline check-promotion-status -p jaescalo.edge.akau.webperf.it ${PIPELINE_ENV}"
      }
    }
    stage ("Promote dev Env") {
      steps {
        echo "Step 3: Promote dev Env. Job: Jenkins_${JOB_NAME}-${BUILD_NUMBER}"
        sh "source ~/.bash_profile 2> /dev/null; akamai pipeline promote -w -p jaescalo.edge.akau.webperf.it -n staging ${PIPELINE_ENV} -m Jenkins_${JOB_NAME}-${BUILD_NUMBER} --emails jaescalo@akamai.com"
      }
    }
    stage ("Check promotion and test dev Env") {
      parallel {
        stage ("Check dev Env Promotion Status") {
          steps {
            echo "Step 4.a:  Check dev Env Promotion Status. Job: Jenkins_${JOB_NAME}-${BUILD_NUMBER}"
            sh "source ~/.bash_profile 2> /dev/null; akamai pipeline check-promotion-status -p jaescalo.edge.akau.webperf.it ${PIPELINE_ENV}"
          }
        }
        stage("Testing dev Env"){
          steps{
            echo "Step 4.b: Test Suite for dev Env"
          }
        }
      }
    }
    stage ("Commit to GitHub master Branch") {
      steps {
        echo "Step 5: Commit changes to master Branch"
        withCredentials([usernamePassword(credentialsId: '5153d0dc-5894-4cbe-9790-bdb840ec1734', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
          script {
            env.ENCODEDUSER=URLEncoder.encode(GIT_USERNAME, "UTF-8")
          }
          sh """
          cd $WORKSPACE/jaescalo.edge.akau.webperf.it
          git add -A
          git commit -m 'Build success. Committing files'
          git push https://${ENCODEDUSER}:${GIT_PASSWORD}@github.com/jaescalo/akaudevops-pipeline.git HEAD:master -f --tags
          """
        }
      }
    }
  }
}
