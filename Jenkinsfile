pipeline{
    agent any
     
    environment{
         
         TAG = "${BUILD_NUMBER}"
    }

    stages{

        // stage('check Files') {
        //     steps {
        //         sh "ls -R"
        //     }
        // }
          stage('Detect Change') {
            steps{
                script {
                    def detectChanges = sh(
                        script: "git diff --name-only HEAD~1 HEAD",
                        returnStdout: true
                    ).trim()

                     echo "detectChanges:\n${detectChanges}"

                    def allService = [
                              'adservice','cartservice','checkoutservice','currencyservice','emailservice','frontend','loadgenerator','paymentservice','productcatalogservice','recommendationservice','shippingservice','shoppingassistantservice'
                         ]
                    env.detectChanges = allService.findAll {
                        service -> detectChanges.contains("src/${service}")
                }.join(',')

                echo "Service to build: ${env.detectChanges}"

                if (!env.detectChanges){
                    echo"no changes detected"
                }
            }
          }
        }

          stage('Docker login') {
            when {
                expression {env.detectChanges != ''}
            }
            steps {
                 script {
                     withCredentials ([usernamePassword (
                        credentialsId: 'DOCKERHUBCRED',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                     )]) {
                         sh 'echo "$DOCKER_PASS" | docker login -u $DOCKER_USER --password-stdin '
                     }
                 }


            }
          }
         stage('Parallel Microservices Build') {
 
              when {
                    expression {env.detectChanges != ''}
                 } 

            matrix {
                axes {
                    axis {
                        name 'src'
                        values 'adservice','cartservice','checkoutservice','currencyservice','emailservice','frontend','loadgenerator','paymentservice','productcatalogservice','recommendationservice','shippingservice','shoppingassistantservice'
                    }
                }

                stages {
                stage("Image Building"){
                    when {
                        expression{
                          env.detectChanges.split(',').contains(src)
                        } 
                    }
                    steps {
                     sh """
                              echo "work on ${src}"
                              docker build -t heyrohhh/${src}:${TAG} ./src/${src}
                              """                             
                        }
                }
    }
                stage("Trivy Scan"){
                     when {
                        expression{
                          env.detectChanges.split(',').contains(src)
                        } 
                    }

                    steps{
                        sh """
                               trivy image --format json --ignore-unfixed -o trivy_report_${src}.json heyrohhh/${src}:${TAG}
                               trivy image --exit-code 0 --severity CRITICAL,HIGH --ignore-unfixed  heyrohhh/${src}:${TAG}
                        """
                    }
                }

                stage('Image Push'){
                     when {
        expression {
            env.detectChanges.split(',').contains(src)
        }
    }
                    steps{
                          sh """
                               docker push heyrohhh/${src}:${TAG}
                          """
                    }
                }
                
                }
                }
             }
            }

    post {
     always { 
        archiveArtifacts artifacts: 'trivy_report_*.json',
          allowEmptyArchive: true
         sh 'docker image prune -f || true'
     }
}


