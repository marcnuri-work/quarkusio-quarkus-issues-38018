# Quarkus: Openshift extension fails to pull images

https://github.com/quarkusio/quarkus/issues/38018

## Environment

Use developer Sandbox since the reproducer requires an OpenShift cluster.

https://console-openshift-console.apps.sandbox-m4.g2pi.p1.openshiftapps.com/add/ns/mnurisan-dev
https://oauth-openshift.apps.sandbox-m4.g2pi.p1.openshiftapps.com/oauth/token/display


## Reproducer

https://github.com/fedinskiy/reproducer/tree/openshift-pull-failure

1. `mvn -Popenshift package -Dquarkus.container-image.group=mnurisan-dev`
2. `oc get events`

```
19s         Normal    Pulling             pod/app-ff8f5d774-j26ph    Pulling image "fd-test/app:1.0.0-SNAPSHOT"
18s         Warning   Failed              pod/app-ff8f5d774-j26ph    Failed to pull image "fd-test/app:1.0.0-SNAPSHOT": rpc error: code = Unknown desc = reading manifest 1.0.0-SNAPSHOT in EDITED:5000/fd-test/app: manifest unknown
18s         Warning   Failed              pod/app-ff8f5d774-j26ph    Error: ErrImagePull
4s          Normal    BackOff             pod/app-ff8f5d774-j26ph    Back-off pulling image "fd-test/app:1.0.0-SNAPSHOT"
4s          Warning   Failed              pod/app-ff8f5d774-j26ph    Error: ImagePullBackOff
```

The following invocations, do work:
```
mvn -Popenshift package
```

```
mvn -Popenshift package -Dquarkus.container-image.group=mnurisan-dev -Dquarkus.platform.version=3.6.4
```

## Analysis

If I inspect the Pod I get the following message:

```
Failed to pull image "mnurisan-dev/app:1.0.0-SNAPSHOT": rpc error: code = Unknown desc = reading manifest 1.0.0-SNAPSHOT in docker.io/mnurisan-dev/app: requested access to the resource is denied
```

For some reason, it seems that it's trying to pull the image from `docker.io` instead of the internal registry.

### 3.6.4 works

```
mvn -Popenshift package -Dquarkus.container-image.group=mnurisan-dev -Dquarkus.platform.version=3.6.4
```

Works because it does generate a **`DeploymentConfig`** with a valid image stream reference.

The snapshot version doesn't work because it generates a **`Deployment`** instead.
This behavior changed after https://github.com/quarkusio/quarkus/pull/37229.


### With no quarkus.container-image.group works

OpenShift seems to correctly resolve the image stream provided that it doesn't have the prepended group on the name.

However, once we prepend the group in the image name of a **`Deployment`**, it stops working.

According to the [documentation](https://docs.openshift.com/container-platform/4.14/openshift_images/using-imagestreams-with-kube-resources.html), you can correctly use ImageStreams by annotating the `Pod` Template with `alpha.image.policy.openshift.io/resolve-names: '*'`

I checked by manually replacing the template labels, and it does seem to work indeed.

## Further actions

Ioannis [says](https://github.com/quarkusio/quarkus/issues/38018#issuecomment-1908300055) that he will give a quick look. No further action is required from our team for now.

### JKube

We have a similar situation at JKube:

[OpenShift Maven|Gradle Plugin should use Deployments by default](https://github.com/eclipse/jkube/issues/1857)

We need to consider finally moving onto Deployments instead of DeploymentConfigs.

We also need to address the annotation problem.

I've updated the issue accordingly.

## Related links

- https://github.com/quarkusio/quarkus/pull/37229
- https://docs.openshift.com/container-platform/4.14/openshift_images/using-imagestreams-with-kube-resources.html
