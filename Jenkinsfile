
pipeline{

    agent any

    tools {
        maven 'M2_HOME'
    }

    environment {
        BRANCH_NAME = 'main'
        GIT_URL = 'https://github.com/roberttemta/geoapp.git'
        GIT_CREDENTIALS = 'GITHUB_CREDENTIAL'
        SONAQUBE_CRED = 'Sonar_Credentials'
        SONAQUBE_INSTALLATION = 'sonar_server'
        APP_NAME = 'geoapp'
        SCANNER_HOME = tool 'sonar_tool'
        QGC_CONDITION = 'false'
        JFROG_CRED = 'jfrog_Crendentials'
        ARTIFACTPATH = 'target/*.jar'
        ARTIFACTORY_URL = 'http://ec2-3-94-62-104.compute-1.amazonaws.com:8081/artifactory'
        REPO = 'geoapp'
        ARTIFACTTARGETPATH = "release_${BUILD_ID}.jar"
        DOCKER_REPO = '042010941106.dkr.ecr.us-east-1.amazonaws.com/geoapp'
        REPO_URL = '042010941106.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REGISTRY_URL = 'https://042010941106.dkr.ecr.us-east-1.amazonaws.com'
        //REPO_URL = '042010941106.dkr.ecr.us-east-1.amazonaws.com/geoapp'
        AWS_REGION = 'us-east-1'
        
    }

   stages{

    stage('git Checkout'){
            steps{
                git branch: "${BRANCH_NAME}", credentialsId: "${GIT_CREDENTIALS}", \
                url: "${GIT_URL}"
            }
    }

    stage('Maven Clean'){
            steps{
                sh 'mvn clean'
            }
    }

    stage('Maven Test'){
            steps{
                sh 'mvn test'
                
            }
    }
    stage('Maven Compile'){
            steps{
                sh 'mvn compile'
            }
    }
        
    stage('Sonarqube Scan'){
            steps{
                withSonarQubeEnv(credentialsId: "${SONAQUBE_CRED}", \
                installationName: "${SONAQUBE_INSTALLATION}" ) {
              sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=${APP_NAME} -Dsonar.projectKey=${APP_NAME} \
                   -Dsonar.java.binaries=. '''
                }
            }
    }

    stage('Quality Gate Check'){
            steps{
              script{
                 waitForQualityGate abortPipeline: "${QGC_CONDITION}", credentialsId: "${SONAQUBE_CRED}" 
              }
            }
    }
        
    stage('Trivy Scan'){
            steps{
                 sh "trivy fs --format table -o maven_dependency.html ."
            }
    }
        
    stage('Package app'){
            steps{
                sh 'mvn package'
                
            }
    }
/*
    stage('Upload Jar to Jfrog'){
            steps{
                withCredentials([usernamePassword(credentialsId: "${JFROG_CRED}", \
                 usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    script {
                        // Define the artifact path and target location
                        //def artifactPath = 'target/*.jar'
                        //def targetPath = "release_${BUILD_ID}.jar"

                        // Upload the artifact using curl
                        sh """
                            curl -u ${ARTIFACTORY_USER}:${ARTIFACTORY_PASSWORD} \
                                 -T ${ARTIFACTPATH} \
                                 ${ARTIFACTORY_URL}/${REPO}/${ARTIFACTTARGETPATH}
                        """
                    }
                }
            }

    }
        */

    stage('Upload Jar to Jfrog') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${JFROG_CRED}",
                 usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    script {
                // Upload the artifact using curl
                sh '''
                    curl -u $ARTIFACTORY_USER:$ARTIFACTORY_PASSWORD \
                         -T $ARTIFACTPATH \
                         ${ARTIFACTORY_URL}/${REPO}/${ARTIFACTTARGETPATH}
                '''
                    }
                 }
            }
    }


    stage('Docker image Build'){
           steps{
              script{
                sh "docker build --no-cache -t ${DOCKER_REPO}:latest ."
                sh "docker build --no-cache -t ${DOCKER_REPO}:${BUILD_ID} ."
              }
           
           }
    }
  
    stage(' Trivy Scan Docker Image'){
         steps{
           sh """trivy image --format table -o docker_image_report.html ${DOCKER_REPO}:${BUILD_ID}"""
  
         }
    }
   
    stage('Push Image To Registry'){
        steps{
          script{
        //def ecr_passwrd=sh(script: "aws ecr-public get-login-password --region 'us-east-1'")
         //sh "docker login --username AWS --password ${ecr_passwrd} public.ecr.aws/g0j7o9l5"   

            sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${REPO_URL}"
            //sh "docker logout ${ECR_REGISTRY_URL}"
            //sh "aws ecr-public get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY_URL}"
            //sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY_URL}"
            sh "docker push ${DOCKER_REPO}:latest "
            sh "docker push ${DOCKER_REPO}:${BUILD_ID} "
          }
       }
    }
  
 }
        
    
}