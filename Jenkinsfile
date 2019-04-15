pipeline {
  agent any
  environment {
          SERVICE_NAME = 'petclinic-development'
  }
  stages{

      stage('Checkout') {
        steps{
          checkout scm
        }
      }

      stage('Build package') {
        steps{
            sh './mvnw package -DskipTests'
        }
      }

      stage('UnitTest') {
        steps {
            if(env.BRANCH_NAME == "development" || env.BRANCH_NAME == "test") {
                sh './mvnw test'
            }
        }

      }

      stage('Docker image') {
        steps{
            if(env.BRANCH_NAME == "development" || env.BRANCH_NAME == "test" || env.BRANCH_NAME == "master" ) {
              def GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()

              if (env.BRANCH_NAME == 'test') {
                  SERVICE_NAME = "petclinic-test"
              }

              if (env.BRANCH_NAME == 'master') {
                  SERVICE_NAME = "petclinic-production"
              }

              def app = docker.build("${SERVICE_NAME}:${GIT_COMMIT}")

              docker.withRegistry('https://145053809521.dkr.ecr.eu-west-1.amazonaws.com', 'ecr:eu-west-1:aws-dev-cred') {
                app.push("${GIT_COMMIT}")
                app.push('latest')
              }

            }
        }
      }

      stage('Deploy') {
        steps{
            if(env.BRANCH_NAME == "development" || env.BRANCH_NAME == "test") {
              sh "aws ecs update-service --cluster ${SERVICE_NAME} --service petclinic-service --region eu-west-1 --force-new-deployment"
              sh "aws ecs wait services-stable --cluster ${SERVICE_NAME} --services petclinic-service --region eu-west-1 "
            }
        }
      }

      stage('Deploy To Prod'){
          when {
            branch 'master'
          }
          steps {
            input message: 'Tests are ok?', ok: 'Deploy to Production'
            sh "aws ecs update-service --cluster ${SERVICE_NAME} --service petclinic-service --region eu-west-1 --force-new-deployment"
            sh "aws ecs wait services-stable --cluster ${SERVICE_NAME} --services petclinic-service --region eu-west-1 "
          }

        }

      stage("Functional Tests"){
        steps{
            if(env.BRANCH_NAME == "test" ) {
              echo("Testinium functional tests TEST environment");
              testiniumExecution failOnTimeout: true, planId: 2979, projectId: 1659, timeoutSeconds: 600
            } else if(env.BRANCH_NAME == "master") {
              echo("Testinium functional tests PROD environment");
              testiniumExecution failOnTimeout: true, planId: 3028, projectId: 1659, timeoutSeconds: 600
            }
        }
      }
  }
}
