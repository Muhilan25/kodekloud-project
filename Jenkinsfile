pipeline {
    agent any
    tools {
        nodejs 'nodejs-22.6.0'
    }
    environment {
        MONGO_URL = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        SCANNER_HOME = tool('sonarqube-scanner')
        ECR_REPO = "072583797351.dkr.ecr.ap-south-1.amazonaws.com/nodeapp"
        IMAGE_NAME = "nodeapp"
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }

    stages{
        stage("install dependencies") {
            steps {
                sh 'npm install --no-audit'
            }
        }

        // stage("Dependency scanning") {
        //     parallel {
        //         stage("NPM Dependency Audit") {
        //             steps {
        //                 sh '''
        //                     npm audit --audit-level=critical
        //                     echo $?
        //                 '''
        //             }
        //         }

        //         stage("OWASP Dependency check") {
        //             steps {
        //                 dependencyCheck additionalArguments: '''
        //                     --scan \'./\'
        //                     --out  \'./\'
        //                     --format  \'ALL\'
        //                     --prettyPrint''', odcInstallation: 'OWASP-DepCheck-10'
        //                 dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true            
        //             }
        //         }
        //     }
        // }

        // stage("unit testing") {
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'mongo-db-cred', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
        //             sh 'npm test'
        //         }
        //         junit allowEmptyResults: true, testResults: 'test-results.xml'
               
        //     }
        // }

        stage("SAST") {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                     ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=Kodekloud-project \
                        -Dsonar.projectKey=Kodekloud-project \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=node_modules/**,coverage/**,dist/**,.git/**
                    '''
                }
            }
        }

        stage("quality gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }



        stage('Docker Build & Push') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-cred'
                ]]) {

                    sh '''
                        aws ecr get-login-password --region ap-south-1 | \
                        docker login --username AWS --password-stdin \
                        072583797351.dkr.ecr.ap-south-1.amazonaws.com

                        docker build -t ${IMAGE_NAME} .

                        docker tag nodeapp:latest \
                        072583797351.dkr.ecr.ap-south-1.amazonaws.com/${IMAGE_NAME}:${IMAGE_TAG}

                        docker push \
                        ${ECR_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage("trivy image") {
            steps {
                sh '''
                    trivy image ${IMAGE_NAME}:${IMAGE_TAG} \
                        --severity LOW,MEDIUM \
                        --exit-code 0 \
                        --quiet \
                        --format json -o trivy-image-MEDIUM-results.json

                    trivy image ${IMAGE_NAME}:${IMAGE_TAG} \
                        --severity HIGH,CRITICAL \
                        --exit-code 1 \
                        --quiet \
                        --format json -o trivy-image-CRITICAL-results.json
                '''
            }
        }


    }
    post {
        always {
             junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'

             publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir:
             './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Depedency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])

             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: true, reportDir: 'trivy-image-MEDIUM-results.json', reportFiles: 'trivy-image-MEDIUM-vul', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}