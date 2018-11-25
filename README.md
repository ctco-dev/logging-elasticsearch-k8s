## charts: elasticsearch, fluent-bit, kibana
### lightweight setup: 1 master, 1 data, 1 client node

### w/o persistence
```bash
helm install -f ./elastic-no-pv-values.yaml --name logging-es stable/elasticsearch

helm install -f ./kibana-values.yaml --name logging-kibana stable/kibana

helm install -f ./fluent-bit-values.yaml --name logging-fluent-bit stable/fluent-bit
```

### w/ local volume persistence
#### requirements:
- kubernetes 1.10.x
- adjust `nodeAffinity` key `kubernetes.io/hostname` in `pv.yaml` to fix `local` pv to specified host
- create folders `/mnt/disks/logging-es-master` and `/mnt/disks/logging-es-data` to satisfy `local` pv 

```bash
kubectl create -f ./pv.yaml

helm install -f ./elastic-values.yaml --name logging-es stable/elasticsearch

helm install stable/kibana --name logging-kibana -f kibana-values.yaml

helm install --name logging-fluent-bit stable/fluent-bit -f fluent-bit-values.yaml
```

### debug
```bash
#template dry run
helm install -f elastic-values.yaml --debug --dry-run --name logging-es stable/elasticsearch

#upgrade chart sample
helm upgrade logging-kibana -f kibana-values.yaml stable/kibana

#test kibana via port forward
kubectl port-forward svc/logging-kibana 8443:443

#test elastic via client (balancer)
kubectl port-forward svc/logging-es-elasticsearch-client 9200:9200
```

### clean up
```bash
helm delete logging-es --purge
kubectl delete pvc,pv -l release=logging-es

helm delete logging-kibana --purge

helm delete logging-fluent-bit --purge
```
