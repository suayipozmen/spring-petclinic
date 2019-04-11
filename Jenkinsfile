node {
  stage 'Checkout'
  checkout scm

  stage 'Build package'
  sh './mvnw package -DskipTests'

  stage('UnitTest') {
    if(env.BRANCH_NAME == "development") {
        sh './mvnw test'
    }
  }

  stage 'Docker build'
  docker.build('petclinic')
 
  stage 'Docker push'
  docker.withRegistry('https://145053809521.dkr.ecr.eu-west-1.amazonaws.com', 'ecr:eu-west-1:aws-dev-cred') {
    docker.image('petclinic').push('latest')
  }


  stage 'deploy'
  sh "aws ecs update-service --cluster petclinic-dev --service petclinic-service --region eu-west-1 --force-new-deployment"

}
