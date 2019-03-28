# guestbook

generated using Luminus version "3.10.40"

## Prerequisites

You will need [Leiningen][1] 2.0 or above installed.

[1]: https://github.com/technomancy/leiningen

## Running

```shell
lein deps
lein run
```

## Tests

```shell
lein test
```

## Semaphore integration
Semaphore config is stored in `.semaphore/semaphore.yml`. The file defines steps that will be run on every push to the repo. Currently, there are 3 jobs:
1. `Setup` – installs dependencies and caches them.
1. `Tests` – runs tests.
1. `Build` – builds an uberjar and caches it for further deployment.

There's also one promotion defined in the config. Promotions are used to describe deployment steps and their configs are stored separately.
In `.semaphore/production-deploy.yml` there's only one job, that restores cached uberjar and lists files in the current directory to demonstrate that the uberjar was successfully restored.
This is just an example. Depending on your deployment procedures, commands described in this job would be different.

## Caching on Semaphore
Semaphore provides the following commands to work with caching:
1. `cache store KEY PATH` – stores the content of `PATH` at specified `KEY`.
1. `cache restore KEY` – restores the content stored at `KEY`.

Since these commands use `tar` under the hood, cached data cannot be restored at absolute path.
For example, caching `/home/semaphore/.m2` will result in a tar archive having `home/semaphore/.m2` path, with the leading `/` omitted.
That's why after restoring caches with absolute paths, they must be moved to their (absolute) paths.
For example, here's how dependencies installed via Leiningen are cached and restored:
```shell
$ cache store lein-deps-$(checksum project.clj) ~/.m2 # ~/.m2 wil be expanded to /home/semaphore/.m2
$ cache restore lein-deps-$(checksum project.clj) # will extract a tar archive to the current folder
$ mv home/semaphore/.m2 ~/.m2
```

### Caching uberjar
Semaphore doesn't overwrite identical keys when caching. If you want to have a static key (like `uberjar-latest`), you will have to remove it every time before calling `cache store`:
```shell
$ cache delete uberjar-latest
$ cache store uberjar-latest target/uberjar/uberjar.jar
```

### Continuous deployment
To make deployments automatically triggered when all jobs finish successfully, add this option to a promotion config:

```yaml
auto_promote_on:
  - result: passed
```