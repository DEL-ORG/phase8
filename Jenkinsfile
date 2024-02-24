pipeline{
    agent any
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
                docker build -t devopseasylearning/revive-ui:${BUILD_NUMBER} .
                '''
            }
        }
        stage('Build catalog') {
            steps {
                sh '''
                cd $WORKSPACE/REVIVE/src/catalog
                docker build -t devopseasylearning/revive-catalog:${BUILD_NUMBER} .
                docker build -t devopseasylearning/revive-catalog-database:${BUILD_NUMBER} -f Dockerfile-db .
                '''
            }
        }
        stage('Build cart') {
            steps {
                sh '''
                cd $WORKSPACE/REVIVE/src/cart
                docker build -t devopseasylearning/revive-cart:${BUILD_NUMBER} .
                docker build -t devopseasylearning/revive-cart-database:${BUILD_NUMBER} -f Dockerfile-dynamodb .
                '''
            }
        }
        stage('Build orders') {
            steps {
                sh '''
                cd $WORKSPACE/REVIVE/src/orders
                docker build -t devopseasylearning/revive-orders:${BUILD_NUMBER} .
                docker build -t devopseasylearning/revive-orders-database:${BUILD_NUMBER} -f Dockerfile-db .
                
                '''
            }
        }
        stage('Build checkout') {
            steps {
                sh '''
                cd $WORKSPACE/REVIVE/src/checkout
                docker build -t devopseasylearning/revive-checkout:${BUILD_NUMBER} .
                docker build -t devopseasylearning/revive-checkout-database:${BUILD_NUMBER} -f Dockerfile-db .
                '''
            }
        }
        stage('Build assets') {
            steps {
                sh '''
                cd $WORKSPACE/REVIVE/src/assets
                docker build -t devopseasylearning/revive-assets:${BUILD_NUMBER} .
                docker build -t devopseasylearning/revive-orders-database-rabbitmq:${BUILD_NUMBER} -f Dockerfile-rabbitmq .
                '''
            }
        }
        stage('Push ui') {
            when{ 
          expression {
            env.GIT_BRANCH == 'origin/phase8' }

            }
            steps {
                sh '''
            docker push devopseasylearning/revive-ui:${BUILD_NUMBER}
        
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
            docker push devopseasylearning/revive-catalog:${BUILD_NUMBER}
            docker push devopseasylearning/revive-catalog-database:${BUILD_NUMBER}
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
            docker push devopseasylearning/revive-cart:${BUILD_NUMBER}
            docker push devopseasylearning/revive-cart-database:${BUILD_NUMBER}
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
            docker push devopseasylearning/revive-orders:${BUILD_NUMBER}
            docker push devopseasylearning/revive-orders-database:${BUILD_NUMBER}
            docker push devopseasylearning/revive-orders-rabbitmq:${BUILD_NUMBER}
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
            docker push devopseasylearning/revive-checkout:${BUILD_NUMBER}
            docker push devopseasylearning/revive-checkout-database:${BUILD_NUMBER}
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
            docker push devopseasylearning/revive-assets:${BUILD_NUMBER}
            docker push devopseasylearning/revive-orders-database-rabbitmq:${BUILD_NUMBER}
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
}


