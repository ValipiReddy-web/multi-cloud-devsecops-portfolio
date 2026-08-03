 #   End-To-End-DevsecOps Pipeline

 ##  production-ready Jenkins file from scratch

 # Production Pipeline
 ```bash
GitHub
 
1. Checkout Source Code
  
2. GitLeaks Secret Scan -->Security Scan -->Passwords,Tokens ,Secrets 
   
3. Build + Unit Test (Maven)
   
4. SonarQube Code Analysis -->
Purpose
Checks
Bugs
Code Smells
Vulnerabilities
Duplicated Code
   
5. Upload JAR to AWS S3
   
6. Docker Build + Trivy Image Scan
   
7. Push Image to AWS ECR
    
8. Helm Deployment (prod namespace)
    
9. Deployment Validation
    
10. Rollout / Rollback
 
11. Email Notification
```

 ## Stage Explanation 

 ```bash
pipeline {

    agent any

    /*******************************
     * Jenkins Tools
     *******************************/
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    /*******************************
     * Pipeline Parameters
     *******************************/
    parameters {

    choice(
        name: 'ACTION',
        choices: ['DEPLOY', 'SWITCH', 'ROLLBACK'],
        description: 'Deployment Action'
    )

    string(
        name: 'IMAGE_TAG',
        defaultValue: 'v1',
        description: 'Enter Docker Image Tag (v1, v2, v3, release-1.0.0, etc.)'
    )

    string(
        name: 'ACTIVE_VERSION',
        defaultValue: 'v1',
        description: 'Enter Active Version (v1, v2, v3)'
    )

}

    /*******************************
     * Environment Variables
     *******************************/
    environment {

        // AWS
        AWS_REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "123456789012"

        // Amazon ECR
        ECR_REPOSITORY = "contido"
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"

        // Amazon S3
        S3_BUCKET = "prod-java-artifacts"

        // Kubernetes
        NAMESPACE = "prod"

        // Helm
        HELM_RELEASE = "contido-app"
        HELM_CHART = "./contido-helm"

        // SonarQube
        SONAR_HOME = tool('sonar-scanner')

        // Email Notification
        EMAIL_TO = "developers@gmail.com,devops@gmail.com"

    }

    stages {

    /**********************************************
     * Stage 1 : Checkout Source Code
     **********************************************/
        stage('Checkout') {

            steps {

                echo "Cloning Application Repository..."

                git branch: 'main',
                url: 'https://github.com/your-org/contido.git'

            }

        }

    /**********************************************
     * Stage 2 : GitLeaks Secret Scan
     **********************************************/
        stage('GitLeaks Scan') {

            steps {

                echo "Scanning Repository for Secrets..."

                sh '''
                gitleaks detect \
                --source . \
                --verbose \
                --exit-code 1
                '''

            }

        }

    /**********************************************
     * Stage 3 : Build & Unit Test
     **********************************************/
        stage('Build & Unit Test') {

            steps {

                echo "Building Java Application..."

                sh '''
                mvn clean package
                '''

            }

        }

    /**********************************************
     * Stage 4 : SonarQube Static Code Analysis
     **********************************************/
        stage('SonarQube Analysis') {

            steps {

                echo "Running SonarQube Analysis..."

                withSonarQubeEnv('sonar') {

                    sh """
                    ${SONAR_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=java-app \
                    -Dsonar.projectName=java-app \
                    -Dsonar.sources=src \
                    -Dsonar.java.binaries=target/classes
                    """
                }

            }

        }


 /**********************************************
     * Stage 5 : Upload Artifact to Amazon S3
     **********************************************/
    stage('Upload Artifact to S3') {

        steps {

            echo "Uploading Java Artifact to S3..."

            sh """
            aws s3 cp target/*.jar \
            s3://${S3_BUCKET}/
            """

        }
    }


    /**********************************************
     * Stage 6 : Docker Build + Trivy Scan
     *
     * Build Docker image and scan security
     * before pushing to ECR
     **********************************************/
    stage('Docker Build & Trivy Scan') {

        steps {

            script {

                echo "Building Docker Image ${IMAGE_TAG}"

                sh """
                docker build \
                -t ${IMAGE_NAME} .
                """


                echo "Running Trivy Security Scan"

                sh """
                trivy image \
                --severity HIGH,CRITICAL \
                --exit-code 1 \
                --no-progress \
                ${IMAGE_NAME}
                """

            }

        }
    }


    /**********************************************
     * Stage 7 : Login to AWS ECR
     **********************************************/
    stage('AWS ECR Login') {

        steps {

            echo "Authenticating with Amazon ECR"

            sh """

            aws ecr get-login-password \
            --region ${AWS_REGION} | \
            docker login \
            --username AWS \
            --password-stdin ${ECR_REGISTRY}

            """

        }
    }


    /**********************************************
     * Stage 8 : Push Image to ECR
     **********************************************/
    stage('Push Docker Image') {

        steps {

            echo "Pushing Image ${IMAGE_TAG} to ECR"

            sh """

            docker push ${IMAGE_NAME}

            """

        }
    }



    /**********************************************
     * Stage 9 : Blue-Green Deployment
     *
     * Example:
     *
     * Deploy v2 alongside v1
     *
     * image.tag=v2
     * activeVersion=v1
     *

     Note : New pods will deployed ,Traffci is routed through v1 only
     **********************************************/
    stage('Deploy Application') {


        when {

            expression {

                params.ACTION == 'DEPLOY'

            }

        }


        steps {


            echo "Deploying New Version"


            sh """

            helm upgrade --install ${HELM_RELEASE} ${HELM_CHART} \
            --namespace ${KUBE_NAMESPACE} \
            --create-namespace \
            -f ${HELM_CHART}/${HELM_VALUES} \
            --set image.tag=${params.IMAGE_TAG} \
            --set activeVersion=${params.ACTIVE_VERSION}

            """

        }

    }




    /**********************************************
     * Stage 10 : Switch Traffic
     *
     * Example:
     *
     * Current Traffic
     * v1
     *
     * Switch
     * v2
     * Note : Switch the traffic new version of deployment v2 application
     **********************************************/
    stage('Switch Traffic') {


        when {

            expression {

                params.ACTION == 'SWITCH'

            }

        }


        steps {


            echo "Switching Traffic"


            sh """

            helm upgrade ${HELM_RELEASE} ${HELM_CHART} \
            --namespace ${KUBE_NAMESPACE} \
            -f ${HELM_CHART}/${HELM_VALUES} \
            --set activeVersion=${params.ACTIVE_VERSION}

            """

        }

    }



    /**********************************************
     * Stage 11 : Rollback Traffic
     *
     * Example:
     *
     * v3 problem
     *
     * Switch back to v2
     *Note : Rollback application not working properly old version
     **********************************************/
    stage('Rollback') {


        when {

            expression {

                params.ACTION == 'ROLLBACK'

            }

        }


        steps {


            echo "Rolling Back Application"


            sh """

            helm upgrade ${HELM_RELEASE} ${HELM_CHART} \
            --namespace ${KUBE_NAMESPACE} \
            -f ${HELM_CHART}/${HELM_VALUES} \
            --set activeVersion=${params.ACTIVE_VERSION}

            """

        }

    }




    /**********************************************
     * Stage 12 : Deployment Validation
     **********************************************/
    stage('Deployment Validation') {


        steps {


            echo "Validating Kubernetes Deployment"


            sh """

            echo "Checking Namespace"

            kubectl get ns ${KUBE_NAMESPACE}



            echo "Checking Pods"

            kubectl get pods \
            -n ${KUBE_NAMESPACE}



            echo "Checking Services"

            kubectl get svc \
            -n ${KUBE_NAMESPACE}



            echo "Checking Rollout Status"

            kubectl rollout status \
            deployment/${HELM_RELEASE} \
            -n ${KUBE_NAMESPACE}

            """

        }

    }


 /**********************************************
     * Post Actions
     *
     * Email notification after pipeline execution
     **********************************************/
    post {


        /******************************************
         * Pipeline Success Notification
         ******************************************/
        success {


            echo "Pipeline Completed Successfully"


            emailext(

                subject: "SUCCESS: ${JOB_NAME} - Build #${BUILD_NUMBER}",


                body: """

Hello Team,

Jenkins pipeline completed successfully.

Application Details
===================

Job Name       : ${JOB_NAME}

Build Number   : ${BUILD_NUMBER}

Build URL      : ${BUILD_URL}


Deployment Details
==================

Environment    : PROD

Namespace      : ${KUBE_NAMESPACE}

Helm Release   : ${HELM_RELEASE}

Action         : ${params.ACTION}

Image Tag      : ${params.IMAGE_TAG}

Active Version : ${params.ACTIVE_VERSION}

Docker Image   : ${IMAGE_NAME}


Status:

Application deployment completed successfully.


Regards,

DevOps Team

""",


                to: "${EMAIL_TO}"

            )

        }



        /******************************************
         * Pipeline Failure Notification
         ******************************************/
        failure {


            echo "Pipeline Failed"


            emailext(

                subject: "FAILED: ${JOB_NAME} - Build #${BUILD_NUMBER}",


                body: """

Hello Team,

Jenkins pipeline execution failed.


Application Details
===================

Job Name      : ${JOB_NAME}

Build Number  : ${BUILD_NUMBER}

Build URL     : ${BUILD_URL}


Deployment Details
==================

Environment   : PROD

Namespace     : ${KUBE_NAMESPACE}

Helm Release  : ${HELM_RELEASE}

Action        : ${params.ACTION}

Image Tag     : ${params.IMAGE_TAG}


Status:

Please check Jenkins console logs for failure details.


Regards,

DevOps Team

""",


                to: "${EMAIL_TO}"

            )

        }



        /******************************************
         * Pipeline Completion
         ******************************************/
        always {


            echo "Pipeline Execution Completed"


        }

    }

}

```
