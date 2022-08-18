pipeline {
    agent any

    environment {

        AWS_ACCESS_KEY_ID     = credentials('MuhanadSinan-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('MuhannadSinan-aws-secret-access-key')

        AWS_S3_BUCKET         = "artefact-bucket-repo"
        AWS_REGION            = "me-south-1"
        ARTIFACT_NAME         = "hello-world.war"
        AWS_EB_APP_NAME       = "java-webapp"
        AWS_EB_APP_VERSION    = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT    = "Javawebapp-env-1"

        SONAR_PROJECT_KEY     = "Java-Maven-Application"
        SONAR_IP              = "15.185.224.95"
        SONAR_TOKEN           = "sqp_e3bd488d35e2c30fd376c40280aba83a5f217f34"

    }

    stages {
        stage('Validate') {
            steps {                
                sh "mvn validate"
                sh "mvn clean"
            }
        }

         stage('Build') {
            steps {                
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {                
                sh "mvn test"
            }

            post {
                always {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }

        stage('Quality Scan'){
            steps {
                sh '''
                mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                    -Dsonar.host.url=http://$SONAR_IP \
                    -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }

        stage('Package') {
            steps {                
                sh "mvn package"
            }

            post {
                success {
                    archiveArtifacts artifacts: '**/target/**.war', followSymlinks: false                   
                }
            }
        }

        stage('Publish artefacts to S3 Bucket') {
            steps {
                sh "aws configure set region $AWS_REGION"
                sh "aws s3 cp ./target/**.war s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"                
            }
        }

        stage('Deploy') {
            steps {
                sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
                sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
            }
        }        
    }
}
