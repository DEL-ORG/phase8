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



        stage('Push ui') {
           when{ 
         expression {
           env.GIT_BRANCH == 'origin/Phase-8' }
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
           env.GIT_BRANCH == 'origin/Phase-8' }
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
           env.GIT_BRANCH == 'origin/Phase-8' }
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
           env.GIT_BRANCH == 'origin/Phase-8' }
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
           env.GIT_BRANCH == 'origin/Phase-8' }
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
           env.GIT_BRANCH == 'origin/Phase-8' }
           }
           steps {
                sh '''
            docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/asset:${BUILD_NUMBER}
            docker push 637423375996.dkr.ecr.us-east-1.amazonaws.com/asset-db:${BUILD_NUMBER}
                '''
            }
        }
  
        stage('Generate compose-file') {
           when{ 
         expression {
           env.GIT_BRANCH == 'origin/Phase-8' }
           }
           steps {
                script {
    withCredentials([
	            string(credentialsId: 'MYSQL_ROOT_PASSWORD', variable: 'MYSQL_ROOT_PASSWORD'),
                string(credentialsId: 'MYSQL_PASSWORD', variable: 'MYSQL_PASSWORD'),
                string(credentialsId: 'DB_PASSWORD', variable: 'DB_PASSWORD'),
                string(credentialsId: 'AWS_ACCESS_KEY', variable: 'AWS_ACCESS_KEY'),
                string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY'),
                string(credentialsId: 'SPRING_DATASOURCE_PASSWORD', variable: 'SPRING_DATASOURCE_PASSWORD'),
                string(credentialsId: 'POSTGRES_PASSWORD', variable: 'POSTGRES_PASSWORD')   
	          ])
              {
                sh'''
cat <<EOF> docker-compose.yml
version: '2.3'
services:
         ui:
           ports:
             - 7777:8080
           environment:
             - JAVA_OPTS=-XX:MaxRAMPercentage=75.0 -Djava.security.egd=file:/dev/urandom
             - SERVER_TOMCAT_ACCESSLOG_ENABLED=true
             - ENDPOINTS_CATALOG=http://catalog:8080
             - ENDPOINTS_CARTS=http://carts:8080
             - ENDPOINTS_ORDERS=http://orders:8080
             - ENDPOINTS_CHECKOUT=http://checkout:8080
             - ENDPOINTS_ASSETS=http://assets:8080
           hostname: ui
           image: 637423375996.dkr.ecr.us-east-1.amazonaws.com/ui:${BUILD_NUMBER}
           restart: always
           mem_limit: 512m
           cap_drop:
             - ALL
           networks:
             - revive
           depends_on:
             - catalog
             - carts
             - orders
             - checkout
             - assets
         catalog:
           hostname: catalog
           ports:
             - "8081:8080"
           image: 637423375996.dkr.ecr.us-east-1.amazonaws.com/catalog:${BUILD_NUMBER}
           restart: always
           environment:
             - DB_ENDPOINT=catalog-db:3306
             - DB_NAME=sampledb
             - DB_USER=catalog_user
             - GIN_MODE=release
             - DB_MIGRATE=true
             - DB_CONNECT_TIMEOUT=5
             - PORT=8080
             - DB_PASSWORD=${DB_PASSWORD}
           mem_limit: 128m
           cap_drop:
             - ALL
           networks:
             - revive
           depends_on:
             - catalog-db
         catalog-db:
           image: 637423375996.dkr.ecr.us-east-1.amazonaws.com/catalog-db:${BUILD_NUMBER}
           hostname: catalog-db
           restart: always
           environment:
             - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
             - MYSQL_ALLOW_EMPTY_PASSWORD=true
             - MYSQL_DATABASE=sampledb
             - MYSQL_USER=catalog_user
             - MYSQL_PASSWORD=${MYSQL_PASSWORD}
           mem_limit: 128m
           networks:
             - revive
         carts:
           hostname: carts
           image: 637423375996.dkr.ecr.us-east-1.amazonaws.com/carts:${BUILD_NUMBER}
           restart: always
           environment:
             - JAVA_OPTS=-XX:MaxRAMPercentage=75.0 -Djava.security.egd=file:/dev/urandom
             - SERVER_TOMCAT_ACCESSLOG_ENABLED=true
             - SPRING_PROFILES_ACTIVE=dynamodb
             - CARTS_DYNAMODB_ENDPOINT=http://carts-db:8000
             - CARTS_DYNAMODB_CREATETABLE=true
             - AWS_ACCESS_KEY=${AWS_ACCESS_KEY}
             - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
           mem_limit: 256m
           cap_drop:
             - ALL
           networks:
             - revive
           depends_on:
             - carts-db
         carts-db:
           image: 637423375996.dkr.ecr.us-east-1.amazonaws.com/carts-db:${BUILD_NUMBER}
           hostname: carts-db
           restart: always
           mem_limit: 256m
           networks:
             - revive
         orders:
           hostname: orders
           image: 637423375996.dkr.ecr.us-east-1.amazonaws.com/orders:${BUILD_NUMBER}
           restart: always
           environment:
             - JAVA_OPTS=-XX:MaxRAMPercentage=75.0 -Djava.security.egd=file:/dev/urandom
             - SERVER_TOMCAT_ACCESSLOG_ENABLED=true
             - SPRING_PROFILES_ACTIVE=rabbitmq
             - SPRING_DATASOURCE_URL=jdbc:postgresql://orders-db:5432/orders
             - SPRING_DATASOURCE_USERNAME=orders_user
             - SPRING_DATASOURCE_PASSWORD=${SPRING_DATASOURCE_PASSWORD}
             - SPRING_RABBITMQ_HOST=rabbitmq
           mem_limit: 512m
           cap_drop:
             - ALL
           networks:
             - revive
           depends_on:
             - rabbitmq
             - orders-db
             - checkout
         orders-db:
           image: 637423375996.dkr.ecr.us-east-1.amazonaws.com/orders-db:${BUILD_NUMBER}
           hostname: orders-db
           restart: always
           security_opt:
             - "no-new-privileges=true"
           environment:
             - reschedule=on-node-failure
             - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
             - POSTGRES_DB=orders
             - POSTGRES_USER=orders_user
           healthcheck:
             test: ["CMD-SHELL", "pg_isready -d orders -U orders_user"]
             interval: 10s
             timeout: 5s
             retries: 30
           mem_limit: 128m
           networks:
             - revive
         checkout:
           image: 637423375996.dkr.ecr.us-east-1.amazonaws.com/checkout:${BUILD_NUMBER}
           hostname: checkout
           restart: always
           read_only: true
           tmpfs:
             - /tmp:rw,noexec,nosuid
           environment:
             - REDIS_URL=redis://checkout-redis:6379
             - ENDPOINTS_ORDERS=http://orders:8080
           mem_limit: 256m
           cap_drop:
             - ALL
           networks:
             - revive
           depends_on:
             - checkout-redis
         checkout-redis:
           image: 637423375996.dkr.ecr.us-east-1.amazonaws.com/checkout-db:${BUILD_NUMBER}
           hostname: checkout-redis
           restart: always
           mem_limit: 128m
           networks:
             - revive
         assets:
           hostname: assets
           environment:
             - PORT=8080
           image: 637423375996.dkr.ecr.us-east-1.amazonaws.com/asset:${BUILD_NUMBER}
           restart: always
           mem_limit: 64m
           cap_drop:
             - ALL
           networks:
             - revive
         rabbitmq:
           image: 637423375996.dkr.ecr.us-east-1.amazonaws.com/asset-db:${BUILD_NUMBER}
           ports:
             - "6001:5672"
             - "15999:15672"
           networks:
             - revive
networks:
  revive:
    driver: bridge
EOF
    '''
}
}
            }
        }
        
        stage('deployment') {
           when{ 
         expression {
           env.GIT_BRANCH == 'origin/Phase-8' }
           }
           steps {
               sh '''
           docker-compose down --remove-orphans || true
           docker stop $(docker ps -q) && docker rm $(docker ps -aq)
           docker-compose up -d
           sleep 10
           docker-compose ps
               '''
           }
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

