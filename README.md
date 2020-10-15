# Install Grafana Custom Operator

Clone the Grafana Operator
```bash
git clone https://github.com/integr8ly/grafana-operator.git
```

Create namespace and apply some manifest
```
oc create namespace grafana
oc project grafana

cd grafana-operator
oc create -f deploy/crds
oc create -f deploy/roles
oc create -f deploy/cluster_roles
oc create -f deploy/operator.yaml
```

Create Service Account to allow Grafana get metrics from prometheus
```bash
oc create sa prometheus-reader
oc adm policy add-cluster-role-to-user view -z prometheus-reader
```

Get prometheus-reader Service Account's Token
```bash
oc serviceaccounts get-token prometheus-reader
```

Create manifest for example datasource.yaml
```yaml
apiVersion: integreatly.org/v1alpha1
kind: GrafanaDataSource
metadata:
  name: grafanadatasource-prometheus
  namespace: grafana
spec:
  name: grafanadatasource-prometheus.yaml
  datasources:
    - name: Prometheus
      type: prometheus
      access: proxy
      url: "https://prometheus-k8s-openshift-monitoring.apps.openshift.podX.io"
      basicAuth: false
      withCredentials: false
      isDefault: true
      version: 1
      editable: true
      jsonData:
        tlsSkipVerify: true
        timeInterval: "5s"
        httpHeaderName1: "Authorization"
      secureJsonData:
        httpHeaderValue1: "Bearer ${TOKEN}"
```

Create object from the manifest
```bash
oc create -f datasource.yaml
```

Create manifest grafana for example grafana-deploy.yaml
```yaml
apiVersion: integreatly.org/v1alpha1
kind: Grafana
metadata:
  name: grafana
  namespace: grafana
spec:
  ingress:
    enabled: True
  config:
    log:
      mode: "console"
      level: "debug"
    security:
      admin_user: "root"
      admin_password: "secret"
    auth:
      disable_login_form: False
      disable_signout_menu: True
    auth.anonymous:
      enabled: False
  dashboardLabelSelector:
    - matchExpressions:
        - {key: app, operator: In, values: [grafana]}
```

Create object from the manifest
```bash
oc create -f grafana-deploy.yaml
```
