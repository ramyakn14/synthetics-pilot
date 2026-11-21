# Day 2: Plan

**Goal:** finish setup by putting the project on GitHub, then get the backend running and producing metrics on the Mac.

## Before you start

```bash
# Open Docker Desktop and wait for the whale icon, then
minikube start
kubectl get nodes     # minikube should be Ready
```

## Part A: Finish Phase 0 (project folder and GitHub)

1. Create the project folder and add these docs to it.
2. Turn it into a Git repository and make the first commit.
3. Create the repository on GitHub and push.

## Part B: Phase 1 begins (the backend)

4. Run Redis on the Mac in Docker.
5. Set up a Python virtual environment and install the backend's packages.
6. Run the backend and test its endpoints with `curl`:
   - `/api/healthz` returns `ok`
   - `POST /api/jobs` returns a job ID
   - `/api/jobs/<id>` shows `done`
7. Open `/metrics` and find the job and queue metrics.
8. Commit and push.

## Done when

- The project is on GitHub with the Day 1 docs.
- You can submit a job to the backend and watch the counters change on its metrics page.

## Coming on Day 3

Build the Flask frontend, connect it to the backend, and test both together.
