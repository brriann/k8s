Compute Resource Quota
We can limit the total sum of compute resources (CPU, memory, etc.) that can be requested in a given Namespace.

apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: myspace
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.nvidia.com/gpu: 4:



Storage Resource Quota
We can limit the total sum of storage resources (PersistentVolumeClaims, requests.storage, etc.) that can be requested.




Object Count Quota
We can restrict the number of objects of a given type (Pods, ConfigMaps, PersistentVolumeClaims, ReplicationControllers, Services, Secrets, etc.).

apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
  namespace: myspace
spec:
  hard:
    configmaps: "10"
    persistentvolumeclaims: "4"
    pods: "4"
    secrets: "10"
    services: "10"
    services.loadbalancers: "2"


LimitRange


  apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-constraint-per-container
  namespace: myspace
spec:
  limits:
  - default:            # default limits
      cpu: 500m
    defaultRequest:     # default requests
      cpu: 500m
    max:                # max defines the highest value of the range
      cpu: "1"
    min:                # min defines the lowest value of the range
      cpu: 100m
    type: Container
