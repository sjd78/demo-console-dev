## What is the ~~problem~~ challenge?

Take a look at the normal console image -- any source maps?  any component names?  any trace data?

(Use the crc console to demonstrate)

When working with dynamic console plugins, it can be frustratingly difficult to run
console with a dev build to see what is going on inside console.  So....

***How can I run console with a dev build that loads my plugin(s)?***

### Attempt 1 - build and run console in the "normal" style...
  1. clone the repo :point_right: `git clone https://github.com/openshift/console.git`
  1. checkout the branch for the version I want to use :point_right: `cd console; git checkout -b _4.12 -t origin/release-4.12`
  1. run `./build.sh`
  1. fail horribly
  1. install a bunch of requirements on my workstation (yuck, but golang is ok, especially for working with operators)
  1. try the build again
  1. watch the frontend build fail horribly
  1. read up (again) on webpack 4, legacy openssl, bad crypto hash algos, upgrading to webpack 5, tweaking package.json files in console to do things "right", realizing that not even Vojtech has this nut cracked, give up and go get ~~a stiff bourbon~~ some more coffee

But ... oh wait ... the dev setup for my plugin has console running in a off-cluster container.  Maybe there is something there!


### Attempt 2 - try using console's Dockerfiles to build a dev mode container image...
  1. go back to my repo clone
  2. `git reset --hard origin/release=4.12` (or similar)
  3. which `Dockerfile` looks ok? :point_right: [this one looks promising](https://github.com/openshift/console/blob/master/Dockerfile)
  4. lets try the base `Dockerfile`, the images used should be setup for good builds:

```sh
podman build -t localhost/console:test -f Dockerfile .
```

  5. watch the container build fail because the base images are not available
  6. message people in "the know" and get pointed at how to maybe somehow get podman to have a access token to the openshift ci registry (Tal even has a document about how to do this, but it is still painful)
  7. try that a few times but then realize it'll probably break all other repos I've been using, and I don't feel like dealing with *that*
  8. give up and go get ~~a stiff bourbon~~ some more coffee


It is a huge annoyance to try to use images out of openshift ci repos!  Like, huge.

But ... oh wait ... maybe it would be easy to adapt the existing Dockerfile to my own and use public images!?


### Attempt 3 - adapt one of console's Dockerfiles to build my own local container with dev mode console...

  1. Humm, let's start with the base Dockerfile and try to work from there [this one looks promising](https://github.com/openshift/console/blob/master/Dockerfile)
  2. Humm, swapping to UBI containers could work :point_right: [registry of UBI containers](https://catalog.redhat.com/software/containers/search?q=ubi&p=1)
  3. Humm, directly cloning the repo in the Dockerfile could work
  1. Look at the console builder image Dockerfile for some clues on versions of things needed :point_right: [Dockerfile.builder](https://github.com/openshift/console/blob/master/Dockerfile.builder)
  4. Humm, the build works ok that way, but....
  5. Now we need the frontend to build dev instead of prod
  6. Right, do it old school downstream like ... apply patches! :point_right: [patches for branches](https://github.com/kubev2v/forklift-console-plugin/tree/main/build/console-dev/patches)
  7. 1 patch to do `dev-once` instead of `build` :point_right: [patch for dev-once](https://github.com/kubev2v/forklift-console-plugin/blob/main/build/console-dev/patches/master/frontend_00_build.patch)
  8. The container build runs, and dev-once runs, and I can use it just like any other console image! :smile:
  9. Yuck...console log is overrun with i18n error messages, another patch to reduce those! :point_right: [patch for i18n logging](https://github.com/kubev2v/forklift-console-plugin/blob/main/build/console-dev/patches/master/frontend_01_mimimal_i18n_logging.patch)
  10. Looking good.  A few refactors and testing against various branches and it all looks to work

#### Why use patches instead of PR to console?

  ...takes too long to PR
  ...patches can be added/removed/tuned as needed
  ...even if PR, would be be able to backport from master to 4.11?  patches can


## What now?

Now, we can create a dev build of console in a container
locally and use it as if it is any other console image.

The advantages?

  - React dev tools now know the real names of all of the components!  It's easy to see where console stops and plugins begin.

  - Source maps and debugging of console code is available!  This makes it easy to trace app start up, plugin loading or even how extensions work.

  - Extra objects are available in window! Now we can look at a few new runtime data points.

