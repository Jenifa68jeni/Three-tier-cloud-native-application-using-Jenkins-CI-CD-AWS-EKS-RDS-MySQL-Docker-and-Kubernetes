# JENKINS PIPELINE SCRIPT (Declarative Pipeline)

```groovy
pipeline {
    agent { label 'slave' }
    tools {
        maven 'maven'
    }
    environment {
        DOCKER_HUB_USER = 'jenkinsdemo'
        DOCKER_HUB_PASS = 'Jenkins@123'
        IMAGE_NAME = 'product-api'
    }
    stages {
        stage('Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/suffi2002/01_product01_api.git'
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    writeFile file: 'Dockerfile', text: '''
                    FROM eclipse-temurin:17-jre-alpine
                    WORKDIR /usr/app
                    COPY target/*.jar ./product_api.jar
                    CMD ["java", "-jar", "product_api.jar"]
                    '''
                    sh "docker build -t ${IMAGE_NAME} ."
                    sh "docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASS}"
                    sh "docker tag ${IMAGE_NAME} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                script {
                    writeFile file: 'deployment.yaml', text: '''
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: product-api-deployment
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: product-api
                      template:
                        metadata:
                          labels:
                            app: product-api
                        spec:
                          containers:
                          - name: product-api
                            image: jenkinsdemo/product-api:latest
                            ports:
                            - containerPort: 8080
                            env:
                            - name: SPRING_DATASOURCE_URL
                              value: jdbc:mysql://jenkins-db.ciopwmndr.ap-south-1.rds.amazonaws.com:3306/jenkins_db
                            - name: SPRING_DATASOURCE_USERNAME
                              value: jenkinsdemo
                            - name: SPRING_DATASOURCE_PASSWORD
                              value: Jenkins@123
                            - name: SPRING_DATASOURCE_DRIVER_CLASS_NAME
                              value: com.mysql.cj.jdbc.Driver
                    '''
                    writeFile file: 'service.yaml', text: '''
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: product-api-service
                    spec:
                      selector:
                        app: product-api
                      ports:
                      - port: 8080
                        targetPort: 8080
                      type: LoadBalancer
                    '''
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                    sh 'kubectl get svc product-api-service'
                }
            }
        }
    }
}
