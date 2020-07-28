# MEAN App - Build and Environment Config with Kustomize

This repo contains the configuration necessary to build a "contact list" MEAN (Mongo, Express, Angular, Node) application and prepare the envioronments (dev and uat, in this case).

It assumes you already have in place:
* A `cicd` project - this is where the `BuildConfig` and `ImageStream` objects will be created.
* Jenkins running in `cicd` with a `node12` Jenkins agent available.  The [cicd setup from this repository has everything you need](https://github.com/demo-thursday/cicd-openshift-jenkins).

## Create the Builds

Run the following command to create four `BuildConfig`s.  Two are source-to-image, two are Jenkins pipelines.  Make sure you run this from the `cicd` project.

```
oc project cicd
oc apply -k overlays/builds
```

## Create the Environments

To create the `mean-app-dev` and `mean-app-uat` projects, run the following:

```
oc apply -k overlays/mean-app-dev
oc apply -k overlays/mean-app-uat
```

## Allow DEV and UAT to Pull from CICD

Finally, you will need to allow the new dev and uat environments to pull images from the `cicd` project.

```
oc policy add-role-to-user system:image-puller system:serviceaccount:mean-app-dev:default -n cicd
oc policy add-role-to-user system:image-puller system:serviceaccount:mean-app-uat:default -n cicd
```

## Optional: Give Jenkins Access to Deploy to DEV and UAT

If you want Jenkins to be able to rollout changed to DEV and UAT, then you will need to give the Jenkins service account in the `cicd` project access to DEV and UAT.

```
oc policy add-role-to-user admin system:serviceaccount:cicd:jenkins -n mean-app-dev
oc policy add-role-to-user admin system:serviceaccount:cicd:jenkins -n mean-app-uat
```

## Debug: ImageStreams

If your cluster doesn't have the NodeJS 12 or Nginx 1.14 base image, you an import them with the following commands:

```
oc project openshift
oc import-image nginx:1.14 --from registry.redhat.io/rhscl/nginx-114-rhel7 --reference-policy='local' --confirm
oc import-image nodejs:12 --from registry.redhat.io/rhscl/nodejs-12-rhel7 --reference-policy='local' --confirm
```