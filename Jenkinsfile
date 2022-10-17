podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      serviceAccountName: jenkins-admin
      containers:
      - name: jnlp
        image: jenkins/inbound-agent:latest
      - name: k8s
        image: alpine/k8s:1.24.6
        command:
        - sleep
        args:
        - 99d        
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: kaniko-secret
        secret:
            secretName: dockerhub
            items:
            - key: .dockerconfigjson
              path: config.json
''') 
{
  node(POD_LABEL) {
    stage('build') {
      git url: 'https://github.com/dclmict/dclm-webcast.git', branch: 'main'      
      container('kaniko') {
        stage('build webcast-app') {
          sh '''
            /kaniko/executor --context `pwd` --destination opeoniye/dclm-webcast:$BUILD_NUMBER
          '''
        }
      }
    }

    stage('deploy') {
      container('k8s') {
        stage('deploy webcast-app') {
          // script {
          //   kubernetesDeploy(enableConfigSubstitution: true, configs: "k8s/webcast.yaml", kubeconfigId: "kubernetes")
          // }
          // def config = readYaml file: "k8s/webcast.yml"
          // config.metadata.name = params.BUILD_NUMBER
          // writeYaml file: "k8s/webcast.yml", data: config
          sh '''
            // pwd && ls
            apk add gettext
            envsubst < k8s/webcast.yml | kubectl apply -f -
            kubectl get deployments/webcast-app -n devops
            echo 'Kindly visit: http://webcast.k8s/'
          '''
        }
      }
    }

    post {
        always {
            slackSend( channel: "#jenkins", color: "good", message: "Jenkins Pipeline")
        }
    }    

  }
}