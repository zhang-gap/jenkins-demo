node('haimaxy-jnlp') {
    stage('Prepare') {
        echo "1.Prepare Stage"
        sh "ssh -fN -L 36353:127.0.0.1:36353 root@172.30.86.124 -i /root/.ssh/wsl"        
        checkout scm
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}"
            }
        }
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build') {
        echo "3.Build Docker Image Stage"
        sh "docker build -t crpi-ixa35oe96frgh26s.cn-beijing.personal.cr.aliyuncs.com/jenkin-demo/pipeline:${build_tag} ."
    }
    stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'dockerAuth', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword} crpi-ixa35oe96frgh26s.cn-beijing.personal.cr.aliyuncs.com"
            sh "docker push crpi-ixa35oe96frgh26s.cn-beijing.personal.cr.aliyuncs.com/jenkin-demo/pipeline:${build_tag}"
        }
    }
    stage('Deploy') {
        echo "5. Deploy Stage"
        if (env.BRANCH_NAME == 'master') {
            input "确认要部署线上环境吗？"
        }
        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
        sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
        sh "kubectl apply -f k8s.yaml --record"
    }
}
