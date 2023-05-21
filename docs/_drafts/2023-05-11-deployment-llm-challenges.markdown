---
layout: post
title:  "Deploying LLM Applications"
date:   2023-05-11 00:00:00 -0500
categories: jekyll update
regenerate: true
---

## Deployment

I deployed the app to Google App Engine using `gcloud app deploy`, which was straightforward. Some work was needed to set up custom DNS and HTTPS for both HTTP and WebSocket traffic.

Here's the app.yaml for the app deployment.
```yaml

runtime: python
env: flex

instance_class: F2

entrypoint: uvicorn fact_checker.main:app --host 0.0.0.0 --port $PORT --workers 2

resources:
  cpu: 2
  memory_gb: 2.0
  disk_size_gb: 20

runtime_config:
  operating_system: "ubuntu22"
  runtime_version: "3.11"

includes:
  - env_variables.yaml
```
I had to split the yaml into 2 parts and add `env_variables.yaml` to `.gitignore` because it has my secret keys in it.



### Deployment

- Setting up the workload identity provider took longer than it should have. The documentation is not very up-to-date. Once set up, things ran smoothly and are presumably secure. It was a valuable learning experience for me, but overkill for this size project.
- Cloud Run is not ideal for this type of application. The image takes a long time to start up, and Slack has a 3-second response limit. As a result, the user initially receives an error message, and then multiple jobs end up running when the message is retried.
- Pull your sentence_transformer model at build time so it's baked into the image. When the application starts, it only needs to load the model (which is much faster than the initial pull).
