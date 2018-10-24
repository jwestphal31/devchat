   pipeline{

     environment { 
        KUBECONFIG = '/home/gce-test/.kube/config'
    }
    agent { label 'node1' }
    stages {
          stage('Checkout'){
            steps{
                checkout scm
            }
          }


          stage('Package Docker Image'){
              steps{
                   sh "docker build -t us.gcr.io/test-bbfc6/jeffsbooks:latest ."
              }
          }

          stage('Push Docker Image to GCR'){
           steps{
                script{
                    docker.withRegistry('https://us.gcr.io', 'gcr:gcr-284') {
                        docker.image('us.gcr.io/test-bbfc6/jeffsbooks').push('latest')
                    }
                }
            }
          }


           stage('Initiate Trend Micro SmartCheck Container Image Assurance Scan'){
             steps{
                sh "pip install -r requirements_jenkins.txt" 
                sh "python3 /home/gce-test/jenkins_plugin.py"
             }
          }


            stage('Deploy to GKE'){
                steps{
                    sh ". /home/gce-test/venv/bin/activate"
                    sh 'kubectl config use-context gke_test-bbfc6_us-west1-a_jeffsbooks-cluster'
                    sh "kubectl plugin ds assign-policy PCI"
                    sh "kubectl --kubeconfig=/home/gce-test/.kube/config delete --namespace=jeffsbooks deployment jeffsbooks"
                    sh "kubectl --kubeconfig=/home/gce-test/.kube/config apply -f jeffsbooks-deployment-gcp.yaml"
                 }
          }

    }

}
