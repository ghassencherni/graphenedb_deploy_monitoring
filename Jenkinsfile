node {

  git 'https://github.com/ghassencherni/mykveks_deploy_monitoring.git'
  withCredentials([usernamePassword(credentialsId: 'aws_credentials', usernameVariable: 'ACCESS_KEY', passwordVariable: 'SECRET_ACCESS')])
{

  if(action == 'Deploy Monitoring') {


    stage('Getting "config"') {
      copyArtifacts filter: 'config', fingerprintArtifacts: true, projectName: 'mykveks_deploy_eks', selector: upstream(fallbackToLastSuccessful: true)
    }
    stage('Deploy Prometheus') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          kubectl create namespace monitoring
          kubectl create secret generic prom-secret-files --from-file=/var/lib/jenkins/secrets/client.pem --from-file=/var/lib/jenkins/secrets/client-key.pem --from-file=/var/lib/jenkins/secrets/ca.crt -n monitoring
          helm install stable/prometheus --name mykveks-prometheus --values mykveks.prometheus.values --namespace monitoring --version $prometheus_chart_version
         """
    }
    stage('Deploy Grafana') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          helm install stable/grafana --name mykveks-grafana --values mykveks.grafana.values --set adminPassword=$grafana_password --namespace monitoring --values grafana.yaml --version $grafana_chart_version 
         """ 
    }

    stage('Get Grafana URL') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          sleep 20 
          export KUBECONFIG=config
          GRAF_URL=http://\$(kubectl get svc --namespace monitoring mykveks-grafana --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
         """
    }
    
    
  }

if(action == 'Destroy Monitoring') {

    stage('Remove Prometheus Helm') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          helm del --purge mykveks-prometheus
          kubectl delete secret prom-secret-files -n monitoring
         """
    }
    stage('Remove Grafana Helm') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          helm del --purge mykveks-grafana
          kubectl delete services/mykveks-grafana -n monitoring
         """
    }
   }
  }
}
