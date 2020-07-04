pipeline {
  /* The environment specifies the credentials required to push my image to dockerhub */
  environment {
    registry = "muditabaid1097/calc"
    registryCredential = 'dockerhub'
    dockerImage = ''
  }
  agent none
  stages {
    /* This stage downloads the maven:3-alpine Docker image and runs this image as a separate container.
      This means that we have two separate Jenkins and Maven containers running locally in Docker.
      The Maven container becomes the agent that Jenkins uses to run your Pipeline project.*/
    stage('Maven') {
      agent {
        docker {
          image 'maven:3-alpine'
          args '-v /root/.m2:/root/.m2'
        }
      }
      stages {
      /* Build stage downloads the artefacts needed to run our Java Program on jenkins. */
        stage('Build') {
          steps {
            sh 'mvn -B -DskipTests clean package'
          }
        }
      /* This is the test stage of our pipeline. It executes the Maven command to run unit tests on our Java project.
         It also generates a JUnit XML report, which is saved to the target/surefire-reports directory . */
       stage('Test') {
          steps {
            sh 'mvn test'
          }
          post {
            always {
                junit 'target/surefire-reports/*.xml'
            }
          }
        }
      }
    }
    /* This is the deliver stage of our pipeline where we build the docker image of our application using our dockerfile
    and then deliver our image to dockerhub. */
    stage('Deliver') {
      agent any
      stages {
        stage('Building our image') {
          steps{
            script {
              dockerImage = docker.build registry + ":$BUILD_NUMBER"
            }
          }
        }
        stage('Deploy our image to dockerhub') {
          steps{
            script {
              docker.withRegistry( '', registryCredential ) {
                dockerImage.push()
              }
            }
          }
        }
        stage('Cleaning up') {
          steps{
              sh "docker rmi $registry:$BUILD_NUMBER"
          }
        }
      }
    }
  }
}
