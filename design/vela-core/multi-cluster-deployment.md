# Enabling Multi-datacenters Deployment for Application

## Background

One example of complex deployment of a single application would be multi-datacenters that come with two or more availibity zone. 

After finished internal testing, application owners want to rollout new features to customers around the world with lowest risk. With that requirements, canary deployment and availabitity zone come into play. Owners want to rollout in one zone first and then expand it later.

## Using KubeVela to compose multi-datacenters application rollout

Let's imaging that we have more than one datacenter across the globe. In this case 3 datacenters which are `us`, `eu` and `asia`.

In this scenario.

- We will start from datacenter `asia` zone `a`.
- Then, we will continue on datacenter `asia` zone `b`.
- After done with datacenter `asia`, wait for manual approval.
- Then continue the rest but only one zone first. (zone `a`)
- Finally, rollout to all the other zones.

Requirements:
- Number of replicas is calculated based on number of clusters.
- Support availability zones. (subsets in the datacenter)
- React on cluster status and re-balance to other clusters if necessary.
- [TBD] Patch resources in datacenter and subsets level.

```yaml
kind: Application
spec:
  policies:
    - name: multi-datacenter-policy
      type: multi-datacenter
      properties:
        engine: ""
        datacenters:
          - name: us
            placement:
              clusterSelector:
                labels:
                  dc: us
            totalReplicas: 200
            subsets: # optional, to support something like availability zone
              - name: a
                placement:
                  clusterSelector:
                    labels:
                      zone: a
                weight: 1
              - name: b
                placement:
                  clusterSelector:
                    labels:
                      zone: b
                weight: 1
          - name: eu
            placement:
              clusterSelector:
                labels:
                  dc: eu
            totalReplicas: 300
            subsets:
              - name: a
                placement:
                  clusterSelector:
                    labels:
                      zone: a
                weight: 1
              - name: b
                placement:
                  clusterSelector:
                    labels:
                      zone: b
                weight: 1
              - name: c
                placement:
                  clusterSelector:
                    labels:
                      zone: c
                weight: 1
          - name: asia
            placement:
              clusterSelector:
                labels:
                  dc: asia
            totalReplicas: 100
            subsets:
              - name: a
                placement:
                  clusterSelector:
                    labels:
                      zone: a
                weight: 1
              - name: b
                placement:
                  clusterSelector:
                    labels:
                      zone: b
                weight: 1
  
  workflow:
    - name: deploy-asia-a
      type: multi-datacenter
      properties:
        policy: multi-datacenter-policy
        datacenters:
          - name: asia
            subsets:
              - name: a
    
    - name: deploy-asia-b
      type: multi-datacenter
      properties:
        policy: multi-datacenter-policy
        datacenters:
          - name: asia
            subsets:
              - name: b

    - name: wait-for-approval
      type: suspend

    - name: deploy-subsets-a
      type: multi-datacenter
      properties:
        policy: multi-datacenter-policy
        datacenters:
          - name: eu
            subsets:
              - name: a
          - name: us
            subsets:
              - name: b
    
    - name: deploy-subsets-b
      type: multi-datacenter
      properties:
        policy: multi-datacenter-policy
        datacenters:
          - name: eu
            subsets:
              - name: b
              - name: c
          - name: us
            subsets:
              - name: b
```


