# Hypnos

Hypnos is an example of a java operator developed "manually"

Hypnos will scale down the resources at a certain frequencies (define by sleep-cron) and scale it up at a certain frequencies (define by wakeup-cron)

You can create Hypnos strategy like this :

```yaml
apiVersion: shyrkaio.github.io/v1alpha1
kind: Hypnos
metadata:
  name: test-011
  #...
spec:
  namespaceTargetedLabel: "io.shyrka.erebus/hypnos=sample010"
  resourceType:
    - "Deployment"
    - "StatefulSet"
  targetedLabel: "io.shyrka.erebus/hypnos=sample"
  # this is for testing regular cron should be
  wakeup-cron: "0/5 * * * *"
  sleep-cron:  "0/2 * * * *"
  load-policy: no-action 
  dry-run: false 
  pause: false 
  comments: "just a simple comments"
```

- <i>namespaceTargetedLabel</i> : define the label used for the targeted namespaces
- <i>resourceType</i> : list that define the resources that will be scale down and up within the targeted namespaces ["Deployment", "StatefulSet", "DeploymentConfig"]
- <i>targetedLabel</i> : define the label used for the targeted resources within the targeted namespaces
- <i>sleep-cron</i> : define the cron frequency of the scale-down
- <i>wakeup-cron</i> : define the cron frequency of the scale-up
- _<i>load-policy</i> : define the action made after an update of the Hypnos strategy (run-sleep-on-change, run-wake-up-on-change, no-action) (not implemented yet)_
- _<i>dry-run</i> : only update the status to see the targeted resources (not implemented yet)_
- _<i>pause</i> : stop the hypnos strategy to be apply (not implemented yet)_
- <i>comments</i> : is for inner comments

To see [more hypnos examples](src/hypnos-scenarios/ReadMe.md)

## Installation

### from build

Retrieve the 'develop' branch
```bash
git clone https://github.com/shyrkaio/shyrka-hypnos-operator && \
cd shyrka-hypnos-operator/ && git checkout develop
```

Set the target registry
```bash
REG_HOST=quay.io
REG_IMG_HOST=$REG_HOST/shyrkaio/erebus-operator
```

Set the target registry into maven configuration for [more details](http://maven.apache.org/settings.html#)
```bash
cat <<'EOF' > settings-sample.xml
<settings>
<servers>
  <server>
    <id>quay.io</id>
    <username>XXXXXXXXXXX</username>
    <password>...........</password>
  </server>
</servers>
</settings>
EOF
clear
```

Create the CRD (Custom Resource Definition)

```bash
kubectl apply -f src/main/resources/hypnos.crd.yaml
```

Build the java application

```bash
mvn package -pl hypnos-operator -Dquarkus.container-image.registry=$REG_HOST 
# test are in progress so you might need
#-Dmaven.test.skip=true
```

Build the container locally and push it to your registry
```bash
mvn k8s:build k8s:push -Pkubernetes \
          -pl hypnos-operator \
          -Djkube.build.strategy=jib \
          -Djkube.docker.registry=$REG_HOST \
          -Djkube.image.name=$REG_IMG_HOST \
          -Djkube.docker.push.registry=$REG_HOST \
          -Dquarkus.container-image.registry=$REG_HOST \
          -Dquarkus.container-image.build=true  
          #-Dmaven.test.skip=true
```

Create the kubernetes resources definition and apply them to your cluster (you are supposed to be logged)

```bash
mvn k8s:resource k8s:apply -Pkubernetes \
             -pl erebus-operator \
             -Djkube.build.strategy=jib \
             -Djkube.docker.registry=$REG_HOST \
             -Djkube.image.name=$REG_IMG_HOST:latest \
             -Djkube.docker.push.registry=$REG_HOST \
             -Dquarkus.container-image.registry=$REG_HOST \
             -Dquarkus.container-image.build=true 
#-Dmaven.test.skip=true
```

you can see operator logs with `kubectl logs -l app=erebus-operator -n shyrka-erebus-operators -f`

You can test your installation with
```
kubectl apply -f erebus-operator/src/hypnos-scenarios/usecase-010/
```

## from helm
@Todo

## from operator hub
@Todo