podTemplate(label: 'mypod', 
    containers: [ containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)],
    volumes: [ hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
) 
{
    node('mypod') {

        checkout scm

        env.DOCKER_API_VERSION="1.23"
        
        sh "git rev-parse --short HEAD > commit-id"

        tag = readFile('commit-id').replace("\n", "").replace("\r", "")
        appName = "hello-kenzan"
        registryHost = "127.0.0.1:30400/"
        imageName = "${registryHost}${appName}:${tag}"
        env.BUILDIMG=imageName  

        stage "Build"
            container('docker')
            {
                  sh "docker build -t ${imageName} -f applications/hello-kenzan/Dockerfile applications/hello-kenzan"
            }
        
        stage "Push"

            sh "docker push ${imageName}"

        stage "Deploy"

            sh "sed 's#127.0.0.1:30400/hello-kenzan:latest#'$BUILDIMG'#' applications/hello-kenzan/k8s/deployment.yaml | kubectl apply -f -"
            sh "kubectl rollout status deployment/hello-kenzan"
    }
}