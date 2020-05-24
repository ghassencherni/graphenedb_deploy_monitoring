node {

  git 'https://github.com/ghassencherni/graphenedb_deploy_monitoring.git'
  withCredentials([usernamePassword(credentialsId: 'aws_credentials', usernameVariable: 'ACCESS_KEY', passwordVariable: 'SECRET_ACCESS')])
{

  if(action == 'Deploy Monitoring') {


    stage('Getting "config"') {
      copyArtifacts filter: 'config', fingerprintArtifacts: true, projectName: 'graphenedb_deploy_eks', selector: upstream(fallbackToLastSuccessful: true)
    }
    stage('Deploy Prometheus') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          helm install stable/prometheus --name graphenedb-prometheus --values graphenedb.prometheus.values --namespace monitoring --version $prometheus_chart_version
         """
    }
    
    stage('Get the Prometheus URL') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          sleep 20
          export KUBECONFIG=config
          PROMO_URL=http://\$(kubectl get svc --namespace prometheus graphenedb-prometheus-server --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
         """
    }
    
    stage('Deploy Grafana') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          helm install stable/grafana --name graphenedb-grafana --values graphenedb.grafana.values --set adminPassword=$grafana_password --namespace monitoring --values grafana.yaml --version $grafana_chart_version 
         """ 
    }

    stage('Get Grafana URL') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          sleep 20 
          export KUBECONFIG=config
          GRAF_URL=http://\$(kubectl get svc --namespace grafana graphenedb-grafana --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
         """
    }
    
    
  }

if(action == 'Destroy Monitoring') {

    stage('Remove Prometheus Helm') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          helm del --purge graphenedb-prometheus
         """
    }
    stage('Remove Grafana Helm') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          helm del --purge graphenedb-grafana
          kubectl delete services/graphenedb-grafana -n grafana
         """
    }
   }
  }
}
