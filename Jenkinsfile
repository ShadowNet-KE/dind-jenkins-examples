pipeline {
    options {
        disableConcurrentBuilds()
    }
    agent {
        dind-1.0.0 {
            defaultContainer 'docker-client'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  name: dind
spec:
  containers:
  - name: docker-client
    image: docker:19.03.1
    command: ['sleep', '99d']
    env:
    - name: DOCKER_HOST
      value: tcp://localhost:2375
  - name: docker-daemon
    image: docker:19.03.1-dind
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ""
    securityContext:
      privileged: true
    volumeMounts:
      - name: cache
        mountPath: /var/lib/docker
  volumes:
  - name: cache
    hostPath:
      path: /tmp
      type: Directory
"""
        }
    }
    environment {
        IMAGE_PUSH_DESTINATION="kyounger/dind-jenkins:k8s-secret-declarative"
    }
    stages {
        stage('Checkout scm') {
            steps {
                checkout scm
            }
        }
        stage('Docker Build') {
            steps {
                container('docker-client') {
                    withCredentials([file(credentialsId: 'docker-credentials', variable: 'DOCKER_CONFIG_JSON')]) {
                        sh '''
                            mkdir -p $HOME/.docker/
                            cp $DOCKER_CONFIG_JSON /$HOME/.docker/config.json
                            ls -al $HOME/.docker/
                            docker version && DOCKER_BUILDKIT=1 docker build --progress plain -t $IMAGE_PUSH_DESTINATION .
                        '''
                        sh '''
                            docker run -t $IMAGE_PUSH_DESTINATION
                            docker push $IMAGE_PUSH_DESTINATION
                        '''
                    }
                }
            }
        }
    }
}
