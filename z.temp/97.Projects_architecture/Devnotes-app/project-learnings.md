DevNotes Project: Setup and Core Learnings

Personal log of learnings, doubts, and answers while building the DevNotes project.

1. Git Commit Standards

Using Conventional Commits for consistency:

feat: New feature

fix: Bug fix

chore: Routine repository maintenance tasks

docs: Documentation changes

ci: CI/CD changes, e.g., ci(jenkins): fix build script

2. Dockerfile Optimization

FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ ./src/
EXPOSE 8000
CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]


slim: A small Debian-based image, just enough to run Python.

WORKDIR /app: Every command after this runs relative to this path.

EXPOSE 8000: This is documentation only, not enforcement. It doesn't actually open a port; it just signals intent to anyone reading the Dockerfile.

CMD [...]: The default command run when the container starts.

Q: Why copy requirements.txt before the source code?
Docker builds images in layers and caches each layer. Copying requirements.txt separately gives it its own cache layer. If requirements.txt hasn't changed, Docker reuses the cached layer and skips reinstalling dependencies, making builds much faster. If src/ and requirements.txt were copied together, any change in src/ would invalidate the cache and force pip install to rerun every time.

Q: Do I need docker-compose to expose a port for a Kubernetes container?
No. docker-compose and Kubernetes are separate runtimes with separate port-exposure mechanisms. Once something runs as a K8s Pod, docker-compose is irrelevant to it.

3. Kubernetes Deployment Anatomy

apiVersion: apps/v1: Specifies the Kubernetes API version being used, which determines valid fields and behaviors for this resource.

Labels vs. Selectors

template.metadata.labels: Every time this Deployment spins up a new Pod, it stamps this key-value label (app: myapp) onto that Pod's metadata.

spec.selector.matchLabels: This is the query rule the Deployment Controller uses. It constantly scans the namespace asking how many Pods are currently wearing the app: myapp label.

┌────────────────────────────────────────────────────────┐
│                 DEPLOYMENT CONTROLLER                  │
│   1. Selector: Looking for pods with 'app: myapp'      │
└───────────────────────────┬────────────────────────────┘
                             │
            ┌────────────────┴────────────────┐
            │ Does it see enough matching     │
            │ Pods right now?                 │
            └────────┬───────────────┬────────┘
                      │               │
                  ( YES )           ( NO )
                      │               │
                      ▼               ▼
              [ Keep Running ]   [ Create new Pod using  ]
                                 [ 'template' blueprint  ]
                                             │
                                             ▼
                                 ┌────────────────────────┐
                                 │        NEW POD         │
                                 │  Metadata Label:       │
                                 │    app: myapp  ◄───────┼── Stamped by template!
                                 └────────────────────────┘
