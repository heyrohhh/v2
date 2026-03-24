


     def serviceConfig =[ 
                                'adservice' : [dockerfile: './src/adservice/Dockerfile'],
                                'cartservice': [dockerfile: './src/cartservice/src/Dockerfile'], // ← special path
                                'checkoutservice' :[dockerfile: './src/checkoutservice/Dockerfile'],
                                'currencyservice' :[dockerfile: './src/currencyservice/Dockerfile'],
                                'emailservice' :[dockerfile: './src/emailservice/Dockerfile'],
                                'frontend' :[dockerfile: './src/frontend/Dockerfile'],
                                'loadgenerator' :[dockerfile: './src/loadgenerator/Dockerfile'],
                                'paymentservice':[dockerfile: './src/paymentservice/Dockerfile'],
                                'productcatalogservice': [dockerfile: './src/productcatalogservice/Dockerfile'],
                                'recommendationservice': [dockerfile: './src/recommendationservice/Dockerfile'],
                                 'shippingservice':[dockerfile: './src/shippingservice/Dockerfile'],
                               'shoppingassistantservice': [dockerfile: './src/shoppingassistantservice/Dockerfile'],
     ]

pipeline{
    agent any
     
    environment{
         DOC_USER = "heyrohh"
         TAG = "${BUILD_NUMBER}"
    }
    stages{

       
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

               
                stage("Image Building"){
                    when {
                        expression{
                          env.detectChanges.split(',').contains(src)
                        } 
                    }
                    steps {
                          script{
                               def config = serviceConfig[src]

                               if(!config) {
                                   error "no config found for service ${src}"
                               }

                          sh """
                                   echo "we are building image for ${src}"
                                   docker build -f ${config.dockerfile} -t ${DOC_USER}/${src}:${TAG}  ./src/${src}
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
                               trivy image --format json --ignore-unfixed -o trivy_report_${src}.json ${DOC_USER}/${src}:${TAG}
                               trivy image --exit-code 0 --severity CRITICAL,HIGH --ignore-unfixed  ${DOC_USER}/${src}:${TAG}
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
                               docker push ${DOC_USER}/${src}:${TAG}
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


