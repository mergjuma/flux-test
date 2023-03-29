## YAML Generation Process
All kubernetes yaml manifests in `bases/polaris` directory were generated one time using helm via the following process:
```
helm repo add fairwinds-stable https://charts.fairwinds.com/stable

helm repo update

mkdir -p /tmp/chart/old/ /tmp/chart/old/raw /tmp/chart/new/ /tmp/chart/new/raw

helm fetch --destination /tmp/chart/old --untar  fairwinds-stable/polaris --version 0.10.1
helm fetch --destination /tmp/chart/new --untar  fairwinds-stable/polaris --version 3.1.1

helm template --output-dir /tmp/chart/old/raw/ --tiller-namespace ops --namespace ops --name polaris  --set dashboard.replicas=2,dashboard.service.type=NodePort /tmp/chart/old/polaris/

helm template --output-dir /tmp/chart/new/raw/ --tiller-namespace ops --namespace ops --name polaris  --set dashboard.replicas=2,dashboard.service.type=NodePort /tmp/chart/new/polaris/


mv /tmp/chart/new/raw/polaris/templates/*.yaml k8s/kustomize/bases/polaris/

sed -i -e '/helm.sh\/chart:/d' -e '/app.kubernetes.io\/managed-by: Tiller/d' $(find k8s/kustomize/bases/polaris/ -type f -name '*.yaml')

```

Rule Exemptions are currently not integrated into helm chart 3.1.1 so they need to be added to `k8s/kustomize/bases/polaris/configmap.yaml` manually. Base exemptions common between all clusters are added to `k8s/kustomize/bases/polaris/configmap.yaml`.

Ingress is implemented by a sepearate kustomization in `ops-apps-shared-ingress`
