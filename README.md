## Enterprise Linux 10 package repository

This repository holds the rpm packages used by the [rockit](https://github.com/rockit-astro) software infrastructure.  It (ab)uses the [GitHub Pages](https://pages.github.com/) and [GitHub Actions](https://docs.github.com/en/free-pro-team@latest/actions) infrastructure to host a free CDN-backed package repository with all our custom packages "in the cloud".  Any files in the `Packages` subdirectory will be exposed to the public internet, so be careful what you commit!

In practice, you shouldn't need to commit to this repository directly.  The scripts in the root of the repository allow the other repositories to automatically push signed RPMs when commits are pushed to their `master`/`main` branches.

### Adding repository to a Rocky Linux 10 machine

Start by installing the epel repository:
```
sudo yum config-manager --set-enabled crb
sudo yum install epel-release
````

Finally, create a repository definition file at `/etc/yum.repos.d/rockit.repo`:

```
[rockit]
name=rockit Observatory Packages
baseurl=https://rockit-astro.github.io/el10/
enabled=1
gpgcheck=1
gpgkey=https://rockit-astro.github.io/el10/RPM-GPG-KEY
priority=98
```

The packages in this repository should now be available for use.

If you update a package and want to install it locally immediately, then you may need to clear the package cache to force `yum` to fetch the latest repository info:
```
sudo yum clean expire-cache
```

### Automating deployment to this repository

Create a workflow with the following:
```yaml
name: "Packaging (el10)"

on:
  push:
    branches:
      - master

jobs:
  build-packages:
    name: "Packaging (el10)"
    uses: rockit-astro/el10/.github/workflows/build-and-push-packages.yml@master
    with:
      build-cmd: make
      packages: "*.rpm"
    secrets: inherit
```
If the project needs additional build dependencies their rpm names can be listed under `with:`&rarr; `build-deps:`.

### Repository maintenance

The maximum disk quota for a single GitHub repository is currently around 1GB.  This repo is not likely to hit that, but if it does, or if package deployment becomes slow due to long checkout times, then the following should fix it:

* Make a local clone of the repository.
* `git rebase -i --root` to start an interactive rebase of the entire repository (note: this will probably be slow, so you might want to do it in batches instead).
* Squash together all the "Push from..." commits so that the last commit is just a giant addition (at that point probably hundreds) of packages.
* Edit the commit to remove the rpm files from old / obsolete package versions.
* Force-push over the repository using `git push origin +master`.

This completely removes the old packages from the git history, reducing the repository size.
