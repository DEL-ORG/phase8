pipeline{
    agent any
    environment {
        AWS_Cred = 'AWS_Cred'
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '7'))
        //skipDefaultCheckout(true)
        disableConcurrentBuilds()
        timeout (time: 60, unit: 'MINUTES')
        timestamps()
     
    }
    //parameters{

   // }
    //environments{

   // }
   stages{
        stage('Login') {
           environment {
	       DOCKERHUB_CREDENTIALS=credentials('Dockerhub-jenkins')
        }
		steps {
			sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
		}
	}
    
        stage('test ui') {
            agent {
             docker {
               image 'devopseasylearning/maven-revive:v1.0.0'
               args '-u root:root'
            }    
        } 
            steps {
                sh '''
            cd REVIVE/src/ui
            mvn test -DskipTests=true
                '''
            }
        
        }
        stage('test catalog') {
            agent {
             docker {
               image 'devopseasylearning/golang02-revive:v1.0.0'
               args '-u 0:0'
            }    
        }
        steps {
                sh '''
            cd REVIVE/src/catalog 
            go test -buildscv=false
                '''
            }
        
    }
        stage('test cart') {
            agent {
             docker {
               image 'devopseasylearning/maven-revive:v1.0.0'
               args '-u root:root'
            }    
        }
        steps {
                sh '''
            cd REVIVE/src/cart
            mvn test -DskipTests=true
                '''
            }
        
    }
        stage('test orders') {
            agent {
             docker {
               image 'devopseasylearning/maven-revive:v1.0.0'
               args '-u root:root'
            }    
        }
        steps {
                sh '''
            cd REVIVE/src/orders
            mvn test -DskipTests=true
                '''
            }
        
    }
    stage('test checkout') {
            agent {
             docker {
               image 'devopseasylearning/nodejs01-revive:v1.0.0'
               args '-u root:root'
            }    
        }
        steps {
                sh '''
            cd REVIVE/src/checkout 
            npm install
                '''
            }
        
    }
    stage('SonarQube analysis') {
            agent {
                docker {
                  image 'devopseasylearning/sonar-scanner-revive:v1.0.0'
                }
               }
               environment {
        CI = 'true'
        scannerHome='/opt/sonar-scanner'
    }
            steps{
                withSonarQubeEnv('sonar') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
    stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        stage('Build ui') {
            steps {
                sh '''
                cd $WORKSPACE/REVIVE/src/ui
                docker build -t 637423375996.dkr.ecr.us-east-1.amazonaws.com/ui:${BUILD_NUMBER} .
                '''
            }
      }
      stage('Build catalog') {
          steps {
              sh '''
              cd $WORKSPACE/REVIVE/src/catalog
              docker build -t 637423375996.dkr.ecr.us-east-1.amazonaws.com/catalog:${BUILD_NUMBER} .
              docker build -t 637423375996.dkr.ecr.us-east-1.amazonaws.com/catalog-db:${BUILD_NUMBER} -f Dockerfile-db .
              '''
          }
      }
      stage('Build cart') {
          steps {
              sh '''
              cd $WORKSPACE/REVIVE/src/cart
              docker build -t 637423375996.dkr.ecr.us-east-1.amazonaws.com/carts:${BUILD_NUMBER} .
              docker build -t 637423375996.dkr.ecr.us-east-1.amazonaws.com/carts-db:${BUILD_NUMBER} -f Dockerfile-dynamodb .
              '''
          }
      }
      stage('Build orders') {
          steps {
              sh '''
              cd $WORKSPACE/REVIVE/src/orders
              docker build -t 637423375996.dkr.ecr.us-east-1.amazonaws.com/orders:${BUILD_NUMBER} .
              docker build -t 637423375996.dkr.ecr.us-east-1.amazonaws.com/orders-db:${BUILD_NUMBER} -f Dockerfile-db .
              
              '''
          }
      }
      stage('Build checkout') {
          steps {
              sh '''
              cd $WORKSPACE/REVIVE/src/checkout
              docker build -t 637423375996.dkr.ecr.us-east-1.amazonaws.com/checkout:${BUILD_NUMBER} .
              docker build -t 637423375996.dkr.ecr.us-east-1.amazonaws.com/checkout-db:${BUILD_NUMBER} -f Dockerfile-db .
              '''
          }
      }
      stage('Build assets') {
          steps {
              sh '''
              cd $WORKSPACE/REVIVE/src/assets
              docker build -t 637423375996.dkr.ecr.us-east-1.amazonaws.com/asset:${BUILD_NUMBER} .
              docker build -t 637423375996.dkr.ecr.us-east-1.amazonaws.com/asset-db:${BUILD_NUMBER} -f Dockerfile-rabbitmq .
              '''
          }
   }
    
           stage('Configure AWS CLI') {          steps {
                script {
                 withCredentials([[
                            $class: 'AmazonWebServicesCredentialsBinding',
                            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                            credentialsId: env.AWS_Cred,
                            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                        ]])                      {   
                    sh 'aws configure set aws_access_key_id ${AWS_ACCESS_KEY_ID}'
                    sh 'aws configure set aws_secret_access_key ${AWS_SECRET_ACCESS_KEY}'
                    sh 'aws configure set default.region "us-east-1"'
                    
                    // Optionally, you can set other configurations such as output format
                    sh 'aws configure set default.output json'
                    sh "aws s3 ls"
                }
            }
            }
        }
        stage('Login ECR') {

    steps {
        script {

                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 637423375996.dkr.ecr.us-east-1.amazonaws.com'
            }
        }
    }
}


        stage('Push ui') {
           when{ 
         expression {
           env.GIT_BRANCH == 'origin/phase8' }
           }
           steps {
               sh '''
           docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/ui:${BUILD_NUMBER}
       
               '''
           }
       }
        stage('Push catalog') {
           when{ 
         expression {
           env.GIT_BRANCH == 'origin/phase8' }
           }
           steps {
               sh '''
           docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/catalog:${BUILD_NUMBER}
           docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/catalog-db:${BUILD_NUMBER}
               '''
           }
       }
        stage('Push cart') {
           when{ 
         expression {
           env.GIT_BRANCH == 'origin/phase8' }
           }
           steps {
               sh '''
           docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/carts:${BUILD_NUMBER}
           docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/carts-db:${BUILD_NUMBER}
               '''
           }
       }
        stage('Push orders') {
           when{ 
         expression {
           env.GIT_BRANCH == 'origin/phase8' }
           }
           steps {
               sh '''
           docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/orders:${BUILD_NUMBER}
           docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/orders-db:${BUILD_NUMBER}
           
               '''
           }
       }
        stage('Push checkout') {
           when{ 
         expression {
           env.GIT_BRANCH == 'origin/phase8' }
           }
           steps {
               sh '''
           docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/checkout:${BUILD_NUMBER}
           docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/checkout-db:${BUILD_NUMBER}
               '''
           }
       }
        stage('Push assets') {
           when{ 
         expression {
           env.GIT_BRANCH == 'origin/phase8' }
           }
           steps {
                sh '''
            docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/asset:${BUILD_NUMBER}
            docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/asset-db:${BUILD_NUMBER}
                '''
            }
        }
  }
  post {
     always {
      success {
            slackSend color: '#2EB67D',
            channel: 'channel to be provided', 
            message: "*Revive Project Build Status*" +
            "\n Project Name: Revive" +
            "\n Job Name: ${env.JOB_NAME}" +
            "\n Build number: ${currentBuild.displayName}" +
            "\n Build Status : *SUCCESS*" +
            "\n Build url : ${env.BUILD_URL}"
        }
        failure {
            slackSend color: '#E01E5A',
            channel: 'channel to be provided',  
            message: "*Revive Project Build Status*" +
            "\n Project Name: Revive" +
            "\n Job Name: ${env.JOB_NAME}" +
            "\n Build number: ${currentBuild.displayName}" +
            "\n Build Status : *FAILED*" +
            "\n Build User : *Tia*" +
            "\n Action : Please check the console output to fix this job IMMEDIATELY" +
            "\n Build url : ${env.BUILD_URL}"
        }
        //unstable {
          //  slackSend color: '#ECB22E',
          //  channel: 'channel to be provided', 
          //  message: "*Revive Project Build Status*" +
          //  "\n Project Name: Revive" +
          //  "\n Job Name: ${env.JOB_NAME}" +
          //  "\n Build number: ${currentBuild.displayName}" +
          //  "\n Build Status : *UNSTABLE*" +
          //  "\n Action : Please check the console output to fix this job IMMEDIATELY" +
          //  "\n Build url : ${env.BUILD_URL}"
        //}  
    }
    }

def aws_credentials() {
sh """    
sudo rm -rf $HOME/.aws || true
sudo mkdir -p $HOME/.aws || true
sudo chown -R jenkins:jenkins $HOME/.aws

cat <<EOF >  $HOME/.aws/credentials
[default]
aws_access_key_id = ${AWS_ACCESS_KEY_ID}
aws_secret_access_key = ${AWS_SECRET_ACCESS_KEY}
EOF

cat <<EOF >  $HOME/.aws/config
[default]
region = "us-east-1"
output = json
EOF
"""
}

