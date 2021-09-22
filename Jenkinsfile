pipeline {

  agent {
    kubernetes {
      yamlFile 'ci/kubernetes-pod.yaml'
    }
  }

  parameters {

    string(
      name: 'CONTAINER_REGISTRY',
      defaultValue: '598971202176.dkr.ecr.us-west-2.amazonaws.com',
      description: 'The container image registry to push images to'
    )

    string(
      name: 'IMAGE_TAG',
      defaultValue: '1.21.0-pd.test-1',
      description: 'The tag to use for building the container image'
    )

    string(
      name: 'BASEIMAGE',
      defaultValue: '598971202176.dkr.ecr.us-west-2.amazonaws.com/3rdparty/ubuntu:20.10',
      description: 'The tag to use for building the container image'
    )

}

environment {

  CONTAINER_REGISTRY="${params.CONTAINER_REGISTRY}"
  IMAGE_TAG="${params.IMAGE_TAG}"
  BASEIMAGE="${params.BASEIMAGE}"

}

  stages {

    stage('Setup') {
      steps {
        container('golang') {
          sh '''
            apt-get update
            apt-get install -y podman pip
            pip3 install awscli==1.19.76
            aws ecr get-login-password --region us-west-2 \
              | podman login --username AWS --password-stdin $CONTAINER_REGISTRY
          '''
        }
      }
    }

    stage('Build binary') {
      steps {
        container('golang') {
          dir('./cluster-autoscaler') {
            sh 'go build -o ./build/cluster-autoscaler-$(go env GOARCH)'
          }
        }
      }
    }

    stage('Run tests') {
      steps {
        container('golang') {
          dir('./cluster-autoscaler') {
            sh 'go test ./...'
          }
        }
      }
    }

    stage('Build container image') {
      steps {
        container('golang') {
          dir('./cluster-autoscaler') {
            sh '''
              podman build \
                -f Dockerfile.amd64 \
                --build-arg=BASEIMAGE=$BASEIMAGE \
                -t $CONTAINER_REGISTRY/3rdparty/cluster-autoscaler:$IMAGE_TAG \
                ./build
            '''
          }
        }
      }
    }

    stage('Push container image') {
      //when { branch 'cluster-autoscaler-release-1.21' }
      steps {
        container('golang') {
          sh 'podman push $CONTAINER_REGISTRY/3rdparty/cluster-autoscaler:$IMAGE_TAG'
        }
      }
    }

  }
}

