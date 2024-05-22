## component removal

```bash
oc delete crd -l app.kubernetes.io/part-of=powerprotect.dell.com
oc delete clusterrolebinding powerprotect:cluster-role-binding
oc delete clusterrolebinding powerprotect:discovery-cluster-rolebinding
oc delete clusterrole powerprotect:discovery-clusterrole
oc delete clusterrole powerprotect:cluster-role
oc delete namespace powerprotect

oc delete crd -l component=velero
oc delete clusterrolebinding -l component=velero
oc delete namespace velero-ppdm
```
[Back to Index](./index.md#ansible-labs-for-bob-the-builder-2024)