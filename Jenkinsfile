pipeline {
     agent any
     stages {
         stage('Build') {
             steps {
                 sh 'echo "Starting build..."'
             }
         }
         stage('Lint HTML') {
              steps {
                  sh 'tidy -q -e *.html'
              }
         } 
         stage('Build Docker Image') {
              steps {
                  sh 'docker build -t capstone .'
              }
         }
         stage('Upload docker image to repository'){
      steps {
        withDockerRegistry([url: '', credentialsId: 'docker']) {
          sh 'docker tag capstone daphneacharles/capstone'
          sh 'docker push daphneacharles/capstone'
        }
      }
    }
    stage('Provision Kubernetes cluster') {
      steps {
        sh 'echo creating EKS cluster'
        withAWS(credentials: 'aws', region: 'us-east-1') {
        sh '''eksctl create cluster \
            --name capstone \
            --region us-east-1 \
            --nodegroup-name capstone \
            --nodes 3 \
            --nodes-min 1 \
            --nodes-max 4 \
            --managed'''
        }
      } 
    }
    stage('Create EKS config') {
      steps {
        withAWS(credentials: 'aws', region: 'us-east-1') {
        sh 'aws eks --region us-east-1 update-kubeconfig --name capstone'
        } 
      }
    }
    stage('Deploy Kubernetes') {
      steps {
        withAWS(credentials: 'aws', region: 'us-east-1') {
        sh 'kubectl apply -f deployment.yml'
        } 
      }
    }
    stage('Confirm Deployment') {
      steps {
        withAWS(credentials: 'aws', region: 'us-east-1') {
        sh 'kubectl get nodes'
        sh 'kubectl get deployment'
        sh 'kubectl get pod -o wide'
        sh 'kubectl get service/capstone'
        } 
      }
    }
    stage('Cleanup') {
      steps {
        echo 'Starting clean up'
        sh 'docker system prune'
      }
    }
     }
}