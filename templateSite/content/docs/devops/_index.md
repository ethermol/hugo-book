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
---
```
[mos@core /store/repo/ignite-doc/hugo-book]$ git commit -am "Started multi-arch site setup, changed contents"
[master 0000696] Started multi-arch site setup, changed contents
 1 file changed, 12 insertions(+)
[mos@core /store/repo/ignite-doc/hugo-book]$ git push origin master                            (master↑1|✔)  4:49PM
Enumerating objects: 13, done.
Counting objects: 100% (13/13), done.
Delta compression using up to 8 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (7/7), 976 bytes | 976.00 KiB/s, done.
Total 7 (delta 4), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (4/4), completed with 4 local objects.
To repo1.github.com:ethermol/hugo-book.git
   b18c217..0000696  master -> master
[mos@core /store/repo/ignite-doc/hugo-book]$ 
```
---
```
[mos@core /store/repo/ignite/docker]$ make build                                               (master|✚3…)  4:59PM
docker buildx build --platform linux/arm,linux/arm64,linux/amd64 --tag pmostert/knoware . --push
WARN[0000] invalid non-bool value for BUILDX_NO_DEFAULT_LOAD:
[+] Building 12.2s (15/15) FINISHED
 => [internal] load build definition from Dockerfile                                                           0.0s
 => => transferring dockerfile: 32B                                                                            0.0s
 => [internal] load .dockerignore                                                                              0.0s
 => => transferring context: 2B                                                                                0.0s
 => [linux/amd64 internal] load metadata for docker.io/library/alpine:3.12                                     2.4s
 => [linux/arm64 internal] load metadata for docker.io/library/alpine:3.12                                     2.4s
 => [linux/arm/v7 internal] load metadata for docker.io/library/alpine:3.12                                    2.5s
 => [linux/amd64 1/4] FROM docker.io/library/alpine:3.12@sha256:c0e9560cda118f9ec63ddefb4a173a2b2a0347082d7df  0.0s
 => => resolve docker.io/library/alpine:3.12@sha256:c0e9560cda118f9ec63ddefb4a173a2b2a0347082d7dff7dc14272e78  0.0s
 => [linux/arm64 1/4] FROM docker.io/library/alpine:3.12@sha256:c0e9560cda118f9ec63ddefb4a173a2b2a0347082d7df  0.0s
 => => resolve docker.io/library/alpine:3.12@sha256:c0e9560cda118f9ec63ddefb4a173a2b2a0347082d7dff7dc14272e78  0.0s
 => CACHED [linux/amd64 2/4] RUN apk update && apk add --no-cache git hugo && hugo new site doclib && cd docl  0.0s
 => CACHED [linux/amd64 3/4] WORKDIR /doclib                                                                   0.0s
 => [linux/arm/v7 1/4] FROM docker.io/library/alpine:3.12@sha256:c0e9560cda118f9ec63ddefb4a173a2b2a0347082d7d  0.0s
 => => resolve docker.io/library/alpine:3.12@sha256:c0e9560cda118f9ec63ddefb4a173a2b2a0347082d7dff7dc14272e78  0.0s
 => CACHED [linux/arm64 2/4] RUN apk update && apk add --no-cache git hugo && hugo new site doclib && cd docl  0.0s
 => CACHED [linux/arm64 3/4] WORKDIR /doclib                                                                   0.0s
 => CACHED [linux/arm/v7 2/4] RUN apk update && apk add --no-cache git hugo && hugo new site doclib && cd doc  0.0s
 => CACHED [linux/arm/v7 3/4] WORKDIR /doclib                                                                  0.0s
 => exporting to image                                                                                         9.6s
 => => exporting layers                                                                                        0.0s
 => => exporting manifest sha256:843577e2c82d26d0f5c394e5d7002682dcda5ac97dd1677b2f7b3fe4485e89fa              0.0s
 => => exporting config sha256:09536cc5683368c5d59d8312ca7feaaf206f77794fa1d0e629469e4a367eb324                0.0s
 => => exporting manifest sha256:368594157108c1dfe28c07eacc75e93daac7a6004202e6cddecd293c1e298f2a              0.0s
 => => exporting config sha256:8ee18a43d79eb1ef296c682d4373d6f6398886d95f551320744842d86d8c5210                0.0s
 => => exporting manifest sha256:2ed7b8f3305dddf275a4484c7b9048168d577bd2905df10ee9ca62f2c3e4cee5              0.0s
 => => exporting config sha256:6dbaa0daa7c3e11f156318ac555578720fef9485ca67355b6a286afa81d6777b                0.0s
 => => exporting manifest list sha256:98fa632e071061291378acf3d8089cb412c7ab0b5e21031a44b6d8ace7bfc995         0.0s
 => => pushing layers                                                                                          2.8s
 => => pushing manifest for docker.io/pmostert/knoware:latest                                                  6.6s

```
## CICD pipeline
Somehow automate the steps above
