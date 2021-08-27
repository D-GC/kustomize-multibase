# Multi-base deployment with kustomize edit set image issue

#### Build single deployment with image tag for production overlays 

`(cd overlays/production/ && kustomize edit set image my-app=*:0.0.1 && kustomize build)`
<details><summary>Output:</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-app-production
spec:
  replicas: 4
  selector:
    matchLabels:
      app: my-app
  template:
    spec:
      containers:
      - image: gcr.io/my-platform/my-app:0.0.1
        name: my-app

```
</details>

#### Build single deployment with image tag for staging overlays

`(cd overlays/staging && kustomize edit set image my-app=*:0.0.1 && kustomize build)`

<details><summary>Output: </summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-app-staging
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    spec:
      containers:
      - image: gcr.io/my-platform/my-app:0.0.1
        name: my-app
```

</details>

**Note:** above 2 commands modifies the [production kustomization.yaml](./overlays/production/kustomization.yaml) and [staging/kustomization.yaml](./overlays/staging/kustomization.yaml) so before running the below command 
to create deployment for production & staging, remove `newTag` from these files
i.e.
```yaml
images:
- name: my-app
  newName: gcr.io/my-platform/my-app
  newTag: 0.0.1
```

#### Build deployment for staging and production with image tag

`(cd overlays/ && kustomize edit set image my-app=*:0.0.1 && kustomize build)`

<details><summary>Output:</summary>
<p>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-app-production
spec:
  replicas: 4
  selector:
    matchLabels:
      app: my-app
  template:
    spec:
      containers:
      - image: gcr.io/my-platform/my-app
        name: my-app
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-app-staging
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    spec:
      containers:
      - image: gcr.io/my-platform/my-app
        name: my-app
```

</p>
</details>

As you can see the image tag is not present in the output

The reason why `kustomize edit set image` doesn't work is because it only updates the tag in current directory.
After running above command if you check the [overlays/kustomization.yaml](./overlays/kustomization.yaml), you'll see the image `newTag` was added.

### Solutions
1. cd into each deployment and the run `kustomize edit set image` and `popd` to run the multibase kustomization.
But it might be too much if there are many deployments of a same app with different names.

2. Use image label and replace it using `sed` command, it will sovle the issue but that's not the kustomize way    

