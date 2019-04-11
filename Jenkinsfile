node {
  def SERVICE_NAME


  stage 'Checkout'
  checkout scm

  stage 'Build package'
  sh './mvnw package -DskipTests'

  stage('UnitTest') {
    if(env.BRANCH_NAME == "dev") {
        sh './mvnw test'
    }
  }

  stage('Docker image') {
    if(env.BRANCH_NAME == "development" || env.BRANCH_NAME == "test" || env.BRANCH_NAME == "master" ) {
      def GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()

      SERVICE_NAME = "petclinic-development"

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

  stage 'deploy'
    sh "aws ecs update-service --cluster ${SERVICE_NAME} --service petclinic-service --region eu-west-1 --force-new-deployment"
}
