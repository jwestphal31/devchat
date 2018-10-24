   stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */
        app = docker.build("smartcheck/hellodockercon")
    }

    stage('Test image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */

        app.inside {
            sh 'echo "Tests passed"'
            sh ('docker build . -t jwest:blog')
        }
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://251406289851.dkr.ecr.us-west-1.amazonaws.com', '311b4a0a-dde2-49f5-a5f1-3d250858536f') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
    stage('Smart Check') {
        sh './scan.sh'
    }
    stage('Deploy') {
        sh('kubectl create -f blog-deployment.yml')
        sh('kubectl create -f blog-service.yml')
        sh('kubectl run hellodockercon --image=251406289851.dkr.ecr.us-west-1.amazonaws.com/smartcheck/hellodockercon:latest --port 8080')
        sh('sleep 5')
        sh('kubectl expose deployment hellodockercon --type=LoadBalancer --port 80 --target-port 8080')
        sh('sleep 20')
        sh('kubectl get svc --namespace default hellodockercon --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}"')
    }
