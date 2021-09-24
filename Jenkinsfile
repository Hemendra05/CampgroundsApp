properties([pipelineTriggers([githubPush()])])

pipeline {

    /* specify nodes for executing */
    agent any


    // setup parameter for docker image tag
    parameters {
        string(name: "imageTag", defaultValue: "latest", description: "This tag is for creating the Docker Image.")
    }

    stages {

        // create docker image
        stage('DockerImage') {
            steps {
                echo "Creating the docker image for the web app by giving Dockerfile."
                sh "sudo podman build -t hemendra05/yelp-camp:${params.imageTag} ."
                sh "sudo podman push hemendra05/yelp-camp:${params.imageTag}"
                echo "Successfully ceated the image and push to the docker repository."
            }
        }

        // deploying the webapp
        stage('KubernetesDeploy') {
            steps {
                echo "Deploying this Application on the Kubernetes Cluster."
                sh "sudo kubectl apply -f deployApp.yml"
                sh "sudo kubectl get deploy"
                sh "sudo kubectl get po"
                sh "sudo kubectl get svc"
                echo "Successfully Deployed the App"
            }
        }
    }
}
