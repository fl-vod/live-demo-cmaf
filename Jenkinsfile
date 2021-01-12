pipeline {
    agent {
        kubernetes {
            // Rather than inline YAML, in a multibranch Pipeline you could use: yamlFile 'jenkins-pod.yaml'
            // Or, to avoid YAML:
            // containerTemplate {
            //     name 'shell'
            //     image 'ubuntu'
            //     command 'sleep'
            //     args 'infinity'
            // }
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    entrypoint:
    - ""
    command:
    - sleep
    args:
    - infinity
  - name: helm
    image: registry.internal.unified-streaming.com/operations/kubernetes/helm-kubectl/trunk:latest
    command:
    - sleep
    args:
    - infinity
  imagePullSecrets:
  - name: gitlab-registry
'''
            // Can also wrap individual steps:
            // container('shell') {
            //     sh 'hostname'
            // }
        }
    }
    environment {
        KUBECONFIG = credentials('kubeconfig')
        REGISTRY_TOKEN = credentials('gitlab-registry-operations')
        REGISTRY_URL = 'registry.internal.unified-streaming.com'
        DOCKER_REPO = 'registry.internal.unified-streaming.com/operations/demo/live-demo-cmaf'
        GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    }
    stages {
        stage('build') {
            steps {
                container('kaniko') {
                    sh 'echo "{\\"auths\\":{\\"$REGISTRY_URL\\":{\\"username\\":\\"$REGISTRY_TOKEN_USR\\",\\"password\\":\\"$REGISTRY_TOKEN_PSW\\"}}}" > /kaniko/.docker/config.json'
                    sh '''
                        cd `pwd`/ffmpeg && \
                        /kaniko/executor \
                            -f `pwd`/Dockerfile \
                            -c `pwd` \
                            --cache=true \
                            --cache-repo=$DOCKER_REPO/cache \
                            --destination $DOCKER_REPO/$BRANCH_NAME:$GIT_COMMIT \
                            --destination $DOCKER_REPO/$BRANCH_NAME:latest \
                    '''
                }
            }
        }
    /* Deploy stage
    stage('deploy') {
        steps {
            container('helm') {
                sh '''
                    export COMMIT=${SVN_COMMIT:-$LAST_STABLE_SVN}
                    helm --kubeconfig $KUBECONFIG \
                        upgrade \
                        --install \
                        --wait \
                        --timeout 300s \
                        --namespace $RELEASE_NAME \
                        --create-namespace \
                        --set licenseKey=$USP_LICENSE_KEY \
                        --set imagePullSecret.username=$REGISTRY_TOKEN_USR \
                        --set imagePullSecret.password=$REGISTRY_TOKEN_PSW \
                        --set imagePullSecret.secretName=gitlab-reg-secret \
                        --set imagePullSecret.registryURL=$REGISTRY_URL \
                        --set image.repository=$DOCKER_REPO/$BRANCH_NAME \
                        --set image.tag=$GIT_COMMIT \
                        --set environment=$BRANCH_NAME \
                        --set env[0].name=REMOTE_STORAGE_URL \
                        --set env[0].value=$REMOTE_STORAGE_URL \
                        --set env[1].name=S3_ACCESS_KEY \
                        --set env[1].value=$S3_ACCESS_KEY \
                        --set env[2].name=S3_SECRET_KEY \
                        --set env[2].value=$S3_SECRET_KEY \
                        --set env[3].name=S3_REGION \
                        --set env[3].value=$S3_REGION \
                        $RELEASE_NAME \
                        ./chart
                '''
            }
        }
    }
    */
  }
}
