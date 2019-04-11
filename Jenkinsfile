node {
  stage 'Checkout'
  checkout scm

  stage 'Build package'
  println "${BRANCH_NAME}"
  sh 'unset MAVEN_CONFIG && ./mvnw package -DskipTests'

  stage 'Docker build'
  docker.build('petclinic')
 
  stage 'Docker push'
  docker.withRegistry('https://145053809521.dkr.ecr.eu-west-1.amazonaws.com', 'ecr:eu-west-1:aws-dev-cred') {
    docker.image('petclinic').push('latest')
  }
}
