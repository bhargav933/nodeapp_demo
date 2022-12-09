pipeline{
    agent any
    
    options{
        timestamps ()
        disableConcurrentBuilds()
    }
    environment{
        DOCKERHUB_CREDENTIALS = credentials('docker-hub')
    }
    stages{
        stage('Checkout'){
            steps{
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'Jenkins-aws', url: 'git@github.com:bhargav933/nodeapp_demo.git']]])
            }
        }
        stage('Docker Build'){
            steps{
                sh 'docker build -t bhargav933/node-app:$BUILD_NUMBER .'
            }
        }
        stage('Docker Login'){
            steps{
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR  --password-stdin'
            }
        }
        stage('Docker Push'){
            steps{
                sh 'docker push bhargav933/node-app:$BUILD_NUMBER'
                sh 'docker logout'
            }
        }
        stage('Deploy'){
            steps{
                echo "Deploying application"
				sh '''
				var=$(kubectl get rs | grep "node-deployment" |awk '{if ($4 == "2" ) print $0}'| awk '{print $1}')
				echo $var >> rs.txt
				'''
				sh "sed -i 's/build_number/${BUILD_NUMBER}/g' deployment.yaml | kubectl apply -f deployment.yaml"
                sh '''
				sleep 60
				var2=$(kubectl get rs | grep "node-deployment" |awk '{if ($4 == "2" ) print $0}'| awk '{print $1}')
				var1=$(cat rs.txt)
				
				if [ $var1 = $var2 ]
                    then
                        echo "replica is not updated "
                            exit 1
                        else
                            echo "Replica set is updated successfully."
                        fi
				'''
                
            }
        }
    }
  		
}
    
