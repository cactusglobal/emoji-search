#!groovy

pipeline {
  agent any

  environment {
    CI = 'true'
    
    //Git commit has to be used for publishing status
    GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
    
    // Reac commit message to directly pull branch on test server after CI
    CUSTOM_PULL_TEST = sh (script: "git log -1 | grep '\\[ci pull-test\\]'", returnStatus: true)
    
    //Github hub credentials configuration
    REPO = 'emoji-search'    
    GIT_ACC = 'shridhars'
    GIT_CRED_TOKEN = 'gitaccesstoken'
    GIT_CREDS = credentials('gitaccesstoken')
    
    //Docker hub credentials configuration
    DOCKER_REG = 'shridharalve/node'
    DOCKER_REG_CREDS = credentials('dockerhub')
  }

  stages {
    stage("CI Stages") {
      //running CI tests in docker container  
      agent {
        docker {
          image 'reactapp-img:latest'
          args '-u root'
          reuseNode true
        }
      }

      stages {
        stage('Prepare') {
          steps {
            githubNotify account: env.GIT_ACC, context: 'continuous-integration/jenkins', credentialsId: env.GIT_CRED_TOKEN, description: '', gitApiUrl: '', repo: env.REPO, sha: env.GIT_COMMIT_HASH, status: 'PENDING', targetUrl: ''
            checkout scm  
            sh """
              echo Working on branch "${env.GIT_BRANCH}" 
              """                  
          }
        }

        //Install js linting, jscpd for static analysis
        stage('Build') {
          steps {
            sh 'npm install'
            sh 'npm install jscpd --save-dev'
            sh 'npm install eslint --save-dev'
            sh 'npm install eslint-plugin-react --save-dev'
            sh 'npm install babel-eslint --save-dev'            
          }
        }

        //Code statis analysis and report collection
        stage('Static Analysis') {
          steps {
            sh 'node_modules/eslint/bin/eslint.js --ext .js -f checkstyle -o linttext.xml src/'
            
            checkstyle canComputeNew: false, canRunOnFailed: true, defaultEncoding: '', healthy: '', pattern: 'linttext.xml', unHealthy: ''

            sh 'node_modules/jscpd/bin/jscpd'

            dry canComputeNew: false, canRunOnFailed: true, defaultEncoding: '', healthy: '', pattern: 'jscpd.xml', unHealthy: ''        
          }
        }

        // unit/component test code and collect coverage reports
        stage('Test') {
          steps {
            sh 'npm test -- --coverage'
          }

          post {
            always {
              step([
                  $class              : "CloverPublisher",
                  cloverReportDir     : "coverage",
                  cloverReportFileName: "clover.xml"
              ])
            }
          }
        }
      }

      //cleanup workspace
      post {
        always {
          sh "chmod -R 777 ."
          cleanWs()
        }
      }
    }   

    //pull to test server 
    stage('Pull to test server') {
      when {
        expression {
          env.GIT_BRANCH != "origin/master"
          env.CUSTOM_PULL_TEST == "0"
        }
      }
      steps {
        sh 'echo Running the AWS boto3 code to pull to test server'
      }
    }

    //Package the build for production if there are changes on branch branch
    stage('Package for Production') {
      when {
        expression {
          env.GIT_BRANCH == 'origin/master'
        }
      }

      steps {
       
        sh 'echo Call script to deploy container on production or just build pull code and perform blue green deployment with aws instances'

        sh """
          git clone https://github.com/shridhars/emoji-search.git emoji-search
        """

        //building docker image with latest code
        sh """
          docker build -t shridharalve/node:reactapp-"${env.BUILD_NUMBER}" -f emoji-search/automation/Dockerfile-production .
        """        

        sh """
          docker login -u="${env.DOCKER_REG_CREDS_USR}" -p="${env.DOCKER_REG_CREDS_PSW}"
        """

        //pushing docker image to dockerhub
        sh """
          docker push shridharalve/node:reactapp-"${env.BUILD_NUMBER}"        
        """
        slackSend color: "good", message: "@here Code has been packaged into the container image with name react-app:${env.BUILD_NUMBER} and pushed to registry"        
      }
    }
  }


  post {
    always {
      cleanWs()
    }
    //Buidl status notification step on slack and github
    success {
      githubNotify account: env.GIT_ACC, context: 'continuous-integration/jenkins', credentialsId: env.GIT_CRED_TOKEN, description: '', gitApiUrl: '', repo: env.REPO, sha: env.GIT_COMMIT_HASH, status: 'SUCCESS', targetUrl: ''

      slackSend color: "good", message: "@here | Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} has passed. \nCheck at ${env.RUN_DISPLAY_URL}"
    }

    failure {
      githubNotify account: env.GIT_ACC, context: 'continuous-integration/jenkins', credentialsId: env.GIT_CRED_TOKEN, description: '', gitApiUrl: '', repo: env.REPO, sha: env.GIT_COMMIT_HASH, status: 'FAILURE', targetUrl: ''

      slackSend color: "danger", message: "@here | Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} has failed. \nCheck at ${env.RUN_DISPLAY_URL}"
    }

    unstable {
      githubNotify account: env.GIT_ACC, context: 'continuous-integration/jenkins', credentialsId: env.GIT_CRED_TOKEN, description: '', gitApiUrl: '', repo: env.REPO, sha: env.GIT_COMMIT_HASH, status: 'ERROR', targetUrl: ''

      slackSend color: "warning", message: "@here | Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} was unstable. \nCheck at ${env.RUN_DISPLAY_URL}"
    }

    aborted {
      githubNotify account: env.GIT_ACC, context: 'continuous-integration/jenkins', credentialsId: env.GIT_CRED_TOKEN, description: '', gitApiUrl: '', repo: env.REPO, sha: env.GIT_COMMIT_HASH, status: 'ERROR', targetUrl: ''

      slackSend color: "warning", message: "@here | Job: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} was aborted. \nCheck at ${env.RUN_DISPLAY_URL}"
    }
  }
}
