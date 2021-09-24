properties([pipelineTriggers([githubPush()])])

pipeline {

    /* specify nodes for executing */
    agent any


    // setup parameter for docker image tag
    parameters {
        string(name: "imageTag", defaultValue: "latest", description: "This tag is for creating the Docker Image.")
    }

    stages {

      // checkout repo
      stage('Checkout SCM') {
          steps {
              checkout([
               $class: 'GitSCM',
               branches: [[name: 'master']],
               userRemoteConfigs: [[
                  url: 'git@github.com:Hemendra05/CampgroundsApp.git',
                  credentialsId: '',
               ]]
            ])
          }
        }

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
                sh "sudo kubectl create deployment yelp-app --image=hemendra05/yelp-camp:latest --port=3000 --replicas=3"
                sh "sudo kubectl expose deployment yelp-app --type=NodePort --port=80 --target-port=3000"
                echo "Successfully Deployed the App"
            }
        }
    }
}
