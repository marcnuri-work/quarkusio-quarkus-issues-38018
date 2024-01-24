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


## WIP

If I inspect the Pod I get the following message:

```
Failed to pull image "mnurisan-dev/app:1.0.0-SNAPSHOT": rpc error: code = Unknown desc = reading manifest 1.0.0-SNAPSHOT in docker.io/mnurisan-dev/app: requested access to the resource is denied
```

For some reason, it seems that it's trying to pull the image from `docker.io` instead of the internal registry.

## Related issues and PRs

- https://github.com/quarkusio/quarkus/pull/37229
- 
