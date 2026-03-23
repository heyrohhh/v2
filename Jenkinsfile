pipeline{
    agent any
     
    environment{
         TAG: "${BUILD_NUMBER}"
    }

    stages{
         stage('Docker Image build') {
            matrix {
                axes {
                    axis {
                        name 'src'
                        values 'adservice','cartservice','checkoutservice','currencyservice','emailservice','frontend','loadgenrator','paymentservice','productcatalogservice','recommendationservice','shippingservice','shoppingassistantservice'
                    }
                }
            }

            stages{
                stage("Building and Pushing"){
                    steps{
                        withCredentials([usernamePassword(
                           credentialsId: 'DOCKERHUBCRED',
                           usernameVariable: 'DOCKER_USER',
                           passwordVariable: 'DOCKER_PASS',
                )]) 
                        script{
                              echo "work on ${src}"
                              sh "docker build -t heyrohhh/${src}:${TAG}"
                              sh "docker push heyrohhh/${src}:{TAG}"
                        }
                    }
                }
            }

         }
    }
}