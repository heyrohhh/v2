pipeline{
    agent any
     
    environment{
         TAG = "${BUILD_NUMBER}"
    }

    stages{

        stage('check Files') {
            steps {
                sh "ls -R"
            }
        }
         stage('Parallel Microservices Build') {
            matrix {

                axes {
                    axis {
                        name 'src'
                        values 'adservice','cartservice','checkoutservice','currencyservice','emailservice','frontend','loadgenerator','paymentservice','productcatalogservice','recommendationservice','shippingservice','shoppingassistantservice'
                    }
                }

                stages {
                stage("Building and Pushing"){
                    steps {
                       
                        script {
                              withCredentials([usernamePassword(
                           credentialsId: 'DOCKERHUBCRED',
                           usernameVariable: 'DOCKER_USER',
                           passwordVariable: 'DOCKER_PASS',
                )]) {

                     sh """
                              echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                              echo "work on ${src}"
                              docker build -t heyrohhh/${src}:${TAG} ./${src}
                              docker push heyrohhh/${src}:${TAG}

                              """

                }
                             
                        }
                    }
                }
             }
            }

         }
    }

    post {
     always {
         sh "docker image prune -f"
     }
}
}

