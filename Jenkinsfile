parameters {
    booleanParam(name: 'FORCE_DEPLOY', defaultValue: true, description: 'Force deployment even if no changes')
}

def serviceConfig = [
    'adservice' : [dockerfile: './src/adservice/Dockerfile'],
    'cartservice': [dockerfile: './src/cartservice/src/Dockerfile',
                    context: './src/cartservice/src'],
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

def serviceMap = [
    'adservice': 'ad-service',
    'cartservice': 'cart-service',
    'checkoutservice': 'checkout-service',
    'currencyservice': 'currency-service',
    'emailservice': 'email-service',
    'frontend': 'frontend-service',
    'loadgenerator': 'load-genrator',
    'paymentservice': 'payment-service',
    'productcatalogservice': 'product-service',
    'recommendationservice': 'recommendation-service',
    'shippingservice': 'shipping-service',
    'shoppingassistantservice': 'shopping-assitant'
]

pipeline {
    agent any

    environment {
        DOC_USER = "heyrohhh"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Detect Change') {
            steps {
                script {
                    def changes = sh(
                        script: "git diff --name-only HEAD~1 HEAD",
                        returnStdout: true
                    ).trim().readLines()

                    def changedServices = serviceConfig.keySet().findAll { svc ->
                        changes.any { it.startsWith("src/${svc}/") }
                    }

                    env.detectChanges = changedServices.join(',')
                    echo "Changed services: ${env.detectChanges}"
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DOCKERHUBCRED',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Build + Push') {
            when {
                expression { env.detectChanges }
            }
            steps {
                script {
                    def services = env.detectChanges.split(',')

                    services.each { svc ->
                        def config = serviceConfig[svc]

                        sh """
                        echo "Building ${svc}"
                        docker build -f ${config.dockerfile} -t ${DOC_USER}/${svc}:${TAG} ${config.context ?: "./src/${svc}"}

                        trivy image --format json --ignore-unfixed -o trivy_${svc}.json ${DOC_USER}/${svc}:${TAG}

                        docker push ${DOC_USER}/${svc}:${TAG}
                        """
                    }
                }
            }
        }

stage('Deploy to Kubernetes') {
    steps {
        script {

            def services = serviceConfig.keySet()

            services.each { svc -> 

                def helmService = serviceMap[svc]

                if (!helmService) {
                    error "No helm mapping for ${svc}"
                }

                sh """
                helm upgrade --install ${helmService} k8s/helm/${helmService} \
                --set image.repository=${DOC_USER}/${svc} \
                --set image.tag=${TAG} \
                --namespace microservice \
                --create-namespace \
                --wait
                """
            }
        }
    }
}
    }

    post {
        always {
            archiveArtifacts artifacts: 'trivy_*.json', allowEmptyArchive: true
            sh 'docker image prune -f || true'
        }
    }
}