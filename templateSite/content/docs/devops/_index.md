---
title: DevOps
weight: 3
---
# DevOps Experience
Will select an hands-on training course to get to the professional level. The index below will be updated while we go.

## Knoware
This is a Hugo template knowledge base. Hugo presents HTML documents after converting them from MarkDown. Using this we have something to work for our DevOps ventures. Before that let's have some content here to start out.
## Building images
The platform executing this application is t4g (graviton 2, AWS ARM64). The docker image will be build on my amd64 based laptop so we should cross-compile the docker image. In order to do this docker buildx (experimental) should be setup and configured. In the end we will execute: *docker buildx build --platform linux/arm,linux/arm64,linux/amd64 --tag pmostert/dmarc . --push* which will create and push (to docker hub) images for amd64 (pc), arm64(pi4, macBook m1) and arm (pi). Kubernetes installed and running on one of these platform should now be able to pull and run the knoware image.
### Steps in the workflow (might change)
* Get remote graviton 2 instance running and install kubernetes.
* Change the site content in the ignite-doc repo and push changes to github.
* Generate new Docker container images and push to Docker hub (public repo).
* Generate new Kubernetes container or pull new site contents from github (public repo)
* Deploy the newly generated docker image on the kubernetes instance
* Check results and have some fun.
```
[mos@core /store/repo/ignite/cfn]$ make create               (master|✚3…)  4:34PM
k3s-stack
k3s-stack.template.yaml
CAPABILITY_NAMED_IAM
{
    "StackId": "arn:aws:cloudformation:eu-central-1:835483671006:stack/k3s-stack/44957b10-3258-11eb-b7eb-0aee1f35785e"
}
```
## CICD pipeline
Somehow automate the steps above
