# Day 28: Creating a Private ECR Repository

### Build the image locally

From the repository root (or wherever **`pyapp/`** lives):

```bash
cd pyapp/
```

```bash
docker build -t pyapp:latest .
```

List images and note the **image ID** (first column in `docker images`; short IDs like `4a395baab5d3` are fine):

```bash
docker images
```

Example (abbreviated):

```text
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
pyapp        latest    4a395baab5d3   2 minutes ago   ...
```

---

### Create a private ECR repository

```bash
aws ecr create-repository --repository-name nautilus-ecr
```

From the JSON response, copy:

- **`repositoryUri`** — form: **`<account-id>.dkr.ecr.<region>.amazonaws.com/nautilus-ecr`**

Example repository URI:

```text
919171157955.dkr.ecr.us-east-1.amazonaws.com/nautilus-ecr
```

ECR repositories are **private by default** unless you change image scanning or repository policies later.

---

### Tag the image for ECR

Point the local image at the ECR URI with a **tag** (commonly **`latest`**). Use **your** image ID and **your** repository URI:

```bash
docker tag 4a395baab5d3 919171157955.dkr.ecr.us-east-1.amazonaws.com/nautilus-ecr:latest
```

**Syntax:** `docker tag <LOCAL_IMAGE_ID_OR_NAME> <repository-uri>:<tag>`

There must be a **colon** before the image tag (`:latest`). A mistaken merge like `.../nautilus-ecrlatest` (no colon) is **invalid** and will not match what you push.

**Alternative:** if you already tagged the build as `pyapp:latest`, you can tag by name instead of numeric ID:

```bash
docker tag pyapp:latest 919171157955.dkr.ecr.us-east-1.amazonaws.com/nautilus-ecr:latest
```

---

### Log in Docker to ECR

Use the **same Region** and **registry host** (account + `dkr.ecr.region.amazonaws.com`) as your repository:

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 919171157955.dkr.ecr.us-east-1.amazonaws.com
```

On success, Docker reports **Login Succeeded**.

---

### Push the image

```bash
docker push 919171157955.dkr.ecr.us-east-1.amazonaws.com/nautilus-ecr:latest
```

---