pipeline {
    agent any

    tools {
        maven "MAVEN"
    }

    stages {
        stage('SonarQube Analysis') {
            steps{
                script{
                    withSonarQubeEnv(installationName: "sonarqube") {
                        bat "mvn clean test sonar:sonar -Dsonar.projectKey=users-microservice-kdl"
                        }
                }

            } 
        }

        stage('Quality Gate Check'){
            steps{
                script{
                    sleep(5)
                        def qg = waitForQualityGate()
                        if(qg.status != 'OK'){
                            error "Pipeline aborted due to quality gate failure"
                    }
                    }
                }
                
            }
        
        stage("Build w/MVN") {
            steps {
                bat "mvn clean package -Dskiptests"
            }
        }

        stage("Build Image w/docker"){
            steps{
                bat "docker build -t $env.AWS_ECR_REGISTRY/users-microservice-kdl:$BUILD_NUMBER ."

            }            
        }

        stage("Deploy to AWS"){
            steps{
                script{
                   docker.withRegistry("https://$env.AWS_ECR_REGISTRY", "ecr:$env.AWS_REGION:AWS"){
                       docker.image("$env.AWS_ECR_REGISTRY/users-microservice-kdl:$BUILD_NUMBER").push()
                   }
                }
            }
        }
            
        stage("Cleaning"){
            steps{
                bat "docker system prune --all -f"
            }
        }
        
    
    }
}