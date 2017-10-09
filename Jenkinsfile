podTemplate(label: 'mypod1', inheritFrom: 'jnlp',
    containers: [ 
        containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
        containerTemplate(
            name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:latest', ttyEnabled: true, command: 'cat',
            envVars: [
                secretEnvVar(key: 'KUBETOKEN', secretName: 'jenkins-token-nlkl6', secretKey: 'token'),
                envVar(key: 'KUBECLUSTER', value: 'https://kubernetes.default'),
                envVar(key: 'KUBECLUSTERNAME', value: 'spc0opj1mg')
            ]
        )
    ],
    volumes: [ hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
) 
{
    node('mypod1') {

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
            container('docker')
            {
                sh "docker push ${imageName}"
            }

        stage "Deploy"
            container('kubectl') {
                sh "kubectl config set-cluster ${env.KUBECLUSTERNAME} --server=${env.KUBECLUSTER} --insecure-skip-tls-verify=true"
                sh "kubectl config set-credentials ${env.KUBECLUSTERNAME} --token=${env.KUBETOKEN}"
                sh "sed 's#127.0.0.1:30400/hello-kenzan:latest#'$BUILDIMG'#' applications/hello-kenzan/k8s/deployment.yaml | kubectl apply -f -"
                sh "kubectl rollout status deployment/hello-kenzan"
            }
    }
}