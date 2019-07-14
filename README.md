# Multi-Repo Orchestration (MRO) Jenkins Pipeline

A Jenkins pipeline to orchestrate multiple repositories into a live application.

## Features

### Automated Resolution of Dependencies

The pipeline automatically resolves dependencies between repositories to be orchestrated so that they can be delivered in the correct order. Currently, repositories that want to be orchestrated need to be added to the `repositories` list inside the `metadata.yml`. Example:

```
repositories:
  - name: my-repo-A
    url: https://github.com/my-org/my-repo-A.git
    branch: refs/heads/master
  - name: my-repo-B
    url: https://github.com/my-org/my-repo-B.git
    branch: refs/heads/master
```

If a named repository wants to announce a dependency on another project to the pipeline, the dependency needs to be listed in that repository's `.pipeline-config.yml`. Example:

```
dependencies:
  - https://github.com/my-org/my-repo-A.git
```

### Automated Parallelization of Repositories

Instead of merely resolving repositories into a strictly sequential execution model, our algorithm automatically understands which repositories are independent and can run in parallel for best time-to-feedback and time-to-delivery.

