# GitLab  Installation 

## Add GitLab Helm Repository

```bash
helm repo add gitlab https://charts.gitlab.io
helm repo update gitlab
   ```
## Install Gitlab Helm Chart

To install GitLab using Helm, run the following command:
```bash
helm install --namespace gitlab gitlab gitlab/gitlab --version 7.9.0 \
  --set global.hosts.domain=ciris.com \
  --set global.edition=ce \
  --set certmanager.install=false \
  --set global.ingress.configureCertmanager=false \
  --set gitlab-runner.install=false
   ```
### Note:
 This installation is done without using a values.yaml file.
## Deploy TLS Secret and Ingress
Create a TLS secret:  <br>
`kubectl create -n gitlab secret tls tls-secret --cert=~/PATH/tls.crt --key=~/PATH/tls.key`

Create an Ingress: <br>


```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gitlab-ingress
  namespace: gitlab
spec:
  tls:
  - hosts:
    - gitlab.ciris.com
    secretName: tls-secret  
  rules:
  - host: gitlab.ciris.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: gitlab-webservice-default
            port:
              number: 8181
   ```

`kubectl apply -f ingress.yaml`

Also, delete the default ingress used by GitLab:  <br>

`kubectl delete ingress ingress-webservice-default -n gitlab`
## Add User after Installation
After installation, you can add a user using the following command: <br>
`curl --request POST --header "PRIVATE-TOKEN: YOUR_ACCESS_TOKEN" --data "email=user@example.com&username=newuser&name=New User&password=strongpassword&confirm=true" "https://gitlab.ciris.com/api/v4/users"`
## Troubleshooting
If you encounter any issues or need to troubleshoot, you can view the entire values.yaml file after your customizations using the following commands: <br>

```bash
helm get values gitlab -n gitlab 
helm show values gitlab/gitlab -n gitlab
   ```
## Applying Configuration Changes

To apply configuration changes after installation, you can use the helm upgrade command along with your updated configuration file (gitlab.yaml): <br>

`helm upgrade gitlab gitlab/gitlab -f gitlab.yaml`

