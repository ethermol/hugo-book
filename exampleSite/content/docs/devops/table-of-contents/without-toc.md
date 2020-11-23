---
title: Without ToC
weight: 2
bookToc: false
---
# DevOps Experience

## Arch workstation
Arch is rolling, arch is current. Arch can be setup and used as a minimalist distribution as such lowering the attack surface. The arch [archwiki](https://wiki.archlinux.org/) is top notch.
1. Create bootable USB
2. Prepare Disks for crypted install
3. Minimal Arch install
4. Add another crypted volume [optional]
5. Install tiling windows manager [spectrwm]
6. Configure stuff

## DevOps-1: Infra and Content
Creating a K8S environment to be managed by CI/CD.
1. Create a Docker-Image containing a simple documentation-site template.
2. Content and Layout for the documentation-site will be stored in a separate repo.
3. Build, Run and test the image locally.
4. Create a multi-arch image and push it to Docker-hub.
5. Within AWS create a t4g (ARM based) instance using cfn
6. Install a single host K8S instance on the t4g-instance.
7. Deploy the Docker-Instance using Helm.

## DevOps-2: CI/CD
To Be Determined (TBD):

> There is an Infra component, software and content so there should be plenty
> opportunity to hook into. This part of the project will be implemented while
> doing the DevOps Professional training. This will compensate for my lack of
> experience in the wild.

