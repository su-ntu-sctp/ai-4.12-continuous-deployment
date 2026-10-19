# Lesson 4.12: Continuous Deployment - Automated Deployment to Railway

## Learning Objectives

By the end of this lesson, you will be able to:
1. **Distinguish** between Continuous Delivery and Continuous Deployment
2. **Deploy** a Docker image from Docker Hub to Railway
3. **Configure** CircleCI to automatically deploy applications after successful tests
4. **Implement** a complete CI/CD pipeline from code push to production

---

## Prerequisites

Before starting this lesson, ensure you have:
- Completed Lesson 4.7 (Continuous Integration with CircleCI)
- Completed Lesson 4.10 (Hosting Options - Manual Hosting)
- Your **devops-demo** GitHub repository from Lesson 4.7, connected to CircleCI
- Docker Hub account with your `devops-demo:latest` image published (public repository)
- CircleCI pipeline showing build ✅ → test ✅ → publish ✅

---

## Introduction

In Lesson 4.7, you built a CI pipeline that automatically builds, tests, and publishes Docker images when you push code to GitHub.

**But there's one problem:** The Docker image sits in Docker Hub. Your application is NOT running anywhere. Users can't access it!

**Today, you'll complete the pipeline:**
- Code push → GitHub
- CircleCI builds, tests, publishes
- **CircleCI tells Railway to deploy the new image** ← NEW!
- Users access your live application

**This is Continuous Deployment (CD)!**

---

## Part 1 - Understanding Continuous Delivery vs Continuous Deployment

### What is Continuous Delivery?

**Continuous Delivery (CD)** is an extension of Continuous Integration. After building and testing, code changes are **automatically prepared** for release to production, but **deployment requires manual approval**.

**Flow:**
```
Code Push → Build → Test → Ready to Deploy → [Manual Button Click] → Production
```

**Example:**
- CircleCI builds and tests successfully
- Deployment package is ready
- Team lead clicks "Deploy to Production" button
- Application goes live

**Why manual approval?**
- Critical applications (banking, healthcare)
- Compliance requirements
- Business timing (deploy during maintenance window)

---

### What is Continuous Deployment?

**Continuous Deployment** goes one step further: **Every** code change that passes tests is **automatically deployed** to production. No manual intervention.

**Flow:**
```
Code Push → Build → Test → Automatically Deploy → Production
```

**Example:**
- Developer pushes code to GitHub
- CircleCI builds and tests
- If tests pass, **automatically** deploys to production
- Users see changes within minutes

**When to use:**
- Startups moving fast
- Non-critical applications
- Teams with strong testing
- DevOps culture

---

### Principles of Continuous Delivery/Deployment

**1. Repeatable Reliable Process**
- Deployment should work the same way every time
- No manual steps that can be forgotten
- Automated deployment scripts

**2. Automate Everything**
- Build, test, deploy - all automated
- No human errors
- Fast and consistent

**3. Version Control Everything**
- Code, configuration, infrastructure
- Can rollback to any previous version
- Track what changed when

**4. Build Quality In**
- Comprehensive tests (unit, integration, E2E)
- Code reviews before merge
- Automated quality checks

**5. Do the Hardest Parts First**
- Deploy frequently (reduces risk)
- Small changes easier to debug
- Don't wait for "big bang" release

**6. Everyone is Responsible**
- Developers own deployment
- No "throw over the wall" to ops
- DevOps culture

**7. "Done" Means Released**
- Feature isn't done until users have it
- Not done when code is merged
- Done when deployed to production

**8. Continuous Improvement**
- Monitor, learn, improve
- Faster deployments over time
- Reduce deployment failures

---

### Benefits of Continuous Deployment

**1. Automate the Software Release Process**
- No manual deployment steps
- Deployment happens in minutes, not days
- Consistent process every time

**2. Improve Developer Productivity**
- Developers focus on features, not deployment
- No waiting for deployment windows
- Fast feedback loop

**3. Find and Address Bugs Quicker**
- Small changes easier to debug
- Issues caught immediately
- Quick rollbacks if needed

**4. Deliver Updates Faster**
- Features reach users immediately
- Competitive advantage
- Rapid iteration based on feedback

---

### CD Pipeline

A typical Continuous Deployment pipeline contains these stages:

**1. Detect New Release**
- Triggered by code push or successful CI
- Identifies what to deploy (Docker image tag, commit hash)

**2. Pull Image from Container Registry**
- Gets the Docker image built by CI
- Verifies image integrity

**3. Deploy to Environment**
- Stops old version
- Starts new version
- Health checks verify deployment

**4. Post-Deployment**
- Run smoke tests
- Monitor for errors
- Notify team of deployment status

---

### Deployment Environments

Professional teams typically have multiple environments:

**Development**
- Developers' local machines
- Rapid changes, may be unstable
- Connected to local/test database

**Staging**
- Mirror of production environment
- User Acceptance Testing (UAT) by stakeholders
- Test with production-like data
- Final verification before production

**Production**
- Live environment where real users access the application
- Must be stable and monitored
- Zero downtime deployments preferred

**Disaster Recovery (DR)**
- Backup environment in different geographic location
- Activated if production fails (earthquake, fire, server failure)
- Keeps business running during disasters

**For this lesson:** We'll deploy directly to **Production** on Railway (simple setup for learning).

---

### Use Case: A/B Testing with Continuous Deployment

**What is A/B Testing?**
- Show two versions of a feature to different users
- Measure which performs better
- Make data-driven decisions

**Example:**
- Version A: Blue "Buy Now" button
- Version B: Green "Buy Now" button
- Deploy both, measure which gets more clicks

**How CD Enables A/B Testing:**
- Change one variable (button color)
- Push code → Automatic deployment
- Users immediately see new version
- Collect metrics quickly
- Deploy winning version to all users

**Without CD:** Would take days/weeks to deploy and test versions.
**With CD:** Deploy and test in hours.

---

## Part 2 - Create a Railway Service from Your Docker Hub Image

In Lesson 4.7, CircleCI built, tested and pushed your image to Docker Hub. Railway will **run that same image** - it does not build anything itself.

```
CircleCI: build → test → publish ──► Docker Hub ──► Railway pulls the image and runs it
```

**Why deploy the image instead of the source code?**
- The image that passed your tests is **exactly** the image that runs in production
- The image is built once, by CI - Railway only pulls and runs it
- Your single-stage Dockerfile from Lesson 4.7 stays as it is

---

### Step 1: Check Your devops-demo Project and Docker Hub Image

Use your **`devops-demo`** repository from Lesson 4.7 - the one connected to CircleCI. You do not need a new repository or any changes to your Dockerfile.

**1.1: Verify the CircleCI pipeline works**

1. Go to [https://app.circleci.com](https://app.circleci.com)
2. Open your devops-demo project
3. Ensure the latest pipeline shows: build ✅ → test ✅ → publish ✅

**If the pipeline is broken, fix it before proceeding!**

**1.2: Check your image on Docker Hub**

1. Go to [https://hub.docker.com](https://hub.docker.com) and open your repositories
2. Find **`devops-demo`** with the tag **`latest`**
3. Make sure the repository is **Public** (Repository → Settings → Visibility)

**Why public?** Railway pulls your image from Docker Hub. Pulling from a private registry requires a paid Railway plan.

Note your full image name - you will need it in Step 3:

```
YOUR_DOCKERHUB_USERNAME/devops-demo:latest
```

---

### Step 2: Create Railway Account

**If you don't have a Railway account from Lesson 4.10:**

1. Go to [https://railway.app](https://railway.app)
2. Click **"Login"** or **"Start a New Project"**
3. Sign up with **GitHub** (recommended)
4. Add a credit card (required but won't charge unless you exceed $5 credit)
5. Verify your email if prompted

**If you already have a Railway account:** Just login.

---

### Step 3: Create a Service from Your Docker Image

**3.1: Create an empty project**

1. Click **"New Project"**
2. Select **"Empty Project"**

**3.2: Add your image as a service**

1. In the project canvas, click **"+ Create"** (or **"Add a Service"**)
2. Select **"Docker Image"**
3. Enter your image name, e.g. `YOUR_DOCKERHUB_USERNAME/devops-demo:latest`, and press **Enter**
4. Click **"Deploy"** (if prompted)

Railway pulls the image from Docker Hub and starts a container. There is **no build step** - the image is already built.

**3.3: Rename the service (optional)**

Click the service → **Settings** → rename it to `devops-demo`.

> **Do not choose "Deploy from GitHub repo".** That makes Railway build and deploy every push by itself - before your tests have run, and outside your CI pipeline. In this lesson, **only CircleCI** decides when to deploy.

---

### Step 4: Generate Public Domain

To access your app from the internet:

1. Click on the **devops-demo** service (in the project view)
2. Click **"Settings"** tab at the top
3. Scroll down to **"Networking"** section
4. Under **"Public Networking"**, click **"Generate Domain"**
5. In the dialog:
   - **Port:** Enter `8080` (Spring Boot default port)
   - Click **"Generate Domain"** button

Railway assigns a URL like:
```
https://devops-demo-production.up.railway.app
```

---

### Step 5: Verify Deployment

**Watch the deployment logs:**

In the Railway dashboard, click the **"Deployments"** tab, then open the latest deployment. You should see the image being pulled (not built), then Spring Boot starting:

```
Pulling image YOUR_DOCKERHUB_USERNAME/devops-demo:latest
...
Started DevopsDemoApplication in X.XXX seconds
```

**Test your application:**

1. Copy the URL from Settings → Networking
2. Add `/hello` to the URL
3. Open in browser:
   ```
   https://devops-demo-production.up.railway.app/hello
   ```

4. You should see: **"DevOps demo application is running!"**

**Success!** Railway is running the image your CI pipeline published.

**But this is not continuous deployment yet.** When you push new code, CircleCI publishes a new image to Docker Hub - and Railway does nothing. Someone would have to redeploy by hand. In Part 3, you make deployment the **last step of the CI pipeline**.

---

## Part 3 - Automate Deployment with CircleCI

### Understanding the Automation Flow

**Current state:**
```
Code Push → GitHub → CircleCI → Build → Test → Publish to Docker Hub → STOPS
```

**What we want:**
```
Code Push → GitHub → CircleCI → Build → Test → Publish → Deploy → LIVE!
```

---

### How CircleCI Triggers Railway

CircleCI cannot "push" an image into Railway. Instead, a new **deploy** job runs after **publish** and tells Railway: *redeploy this service*. Railway then pulls the latest image from Docker Hub and replaces the running container.

**How it works:**
1. CircleCI builds, tests and publishes the new image to Docker Hub
2. If everything passed, the **deploy** job runs `railway redeploy` using the Railway CLI
3. The CLI calls Railway's API, authenticated with a **project token**
4. Railway pulls `devops-demo:latest` from Docker Hub and restarts the service

If **build**, **test** or **publish** fails, the deploy job never runs - broken code never reaches production.

---

### Step 1: Create a Railway Project Token

A **project token** can only deploy to one project and environment - safer for CI than a token for your whole account.

1. Open your Railway project
2. Click **"Settings"** (project settings, not service settings)
3. Click the **"Tokens"** tab
4. Click **"Create Token"**:
   - **Name:** `CircleCI Deployment`
   - **Environment:** `production`
5. **Copy the token** - you'll need it in CircleCI!

**Important:** Save this token securely - it won't be shown again!

---

### Step 2: Get Your Railway Service ID

1. Click on your **devops-demo** service
2. Click **"Settings"** tab
3. Find **"Service ID"**
4. Copy it (format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)

You don't need the project ID - the project token already identifies the project and environment.

---

### Step 3: Add Railway Credentials to CircleCI

**3.1: Go to CircleCI**

1. Open [https://app.circleci.com](https://app.circleci.com)
2. Go to your devops-demo project
3. Click **"Project Settings"**
4. Click **"Environment Variables"**

**3.2: Add Railway Credentials**

Add these two environment variables:

**Variable 1: Project Token**
- **Name:** `RAILWAY_TOKEN`
- **Value:** Your Railway project token from Step 1
- Click **"Add Environment Variable"**

**Variable 2: Service ID**
- **Name:** `RAILWAY_SERVICE_ID`
- **Value:** Your Service ID from Step 2
- Click **"Add Environment Variable"**

**Why `RAILWAY_TOKEN`?** The Railway CLI automatically looks for a project token in the `RAILWAY_TOKEN` environment variable - no `railway login` needed.

**Why environment variables?**
- Keep credentials secret
- Don't expose in code
- Easy to change if needed

---

### Step 4: Update CircleCI Configuration

**4.1: Open Your Project**

Open your devops-demo project in VS Code (or your editor).

**4.2: Edit .circleci/config.yml**

Open `.circleci/config.yml` file.

**Your current config should look like this (from Lesson 4.7):**

```yml
version: 2.1

jobs:
  build:
    docker:
      - image: cimg/openjdk:21.0
    steps:
      - checkout
      - restore_cache:
          keys:
            - maven-deps-{{ checksum "pom.xml" }}
            - maven-deps-
      - run:
          name: Install dependencies and build
          command: |
            echo "Building the application..."
            mvn clean install -DskipTests
      - save_cache:
          paths:
            - ~/.m2
          key: maven-deps-{{ checksum "pom.xml" }}
      - persist_to_workspace:
          root: .
          paths:
            - target/*.jar

  test:
    docker:
      - image: cimg/openjdk:21.0
    steps:
      - checkout
      - restore_cache:
          keys:
            - maven-deps-{{ checksum "pom.xml" }}
            - maven-deps-
      - run:
          name: Run tests
          command: |
            echo "Running tests..."
            mvn test
      - store_test_results:
          path: target/surefire-reports
      - store_artifacts:
          path: target/surefire-reports

  publish:
    docker:
      - image: cimg/base:stable
    steps:
      - checkout
      - attach_workspace:
          at: .
      - setup_remote_docker
      - run:
          name: Build Docker image
          command: |
            echo "Building Docker image..."
            docker build -t $DOCKER_USERNAME/devops-demo:latest .
      - run:
          name: Push to Docker Hub
          command: |
            echo "Logging in to Docker Hub..."
            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
            echo "Pushing image to Docker Hub..."
            docker push $DOCKER_USERNAME/devops-demo:latest

workflows:
  build_test_publish:
    jobs:
      - build
      - test:
          requires:
            - build
      - publish:
          requires:
            - test
```

**4.3: Add Deploy Job**

Add this new `deploy` job after the `publish` job:

```yml
  # ==========================================
  # DEPLOY JOB - Tell Railway to redeploy
  # ==========================================
  deploy:
    docker:
      - image: cimg/node:lts
    steps:
      - run:
          name: Redeploy latest image on Railway
          command: |
            echo "Asking Railway to redeploy the latest image..."
            npx --yes @railway/cli redeploy --service "$RAILWAY_SERVICE_ID" --yes
            echo "Railway is now pulling $DOCKER_USERNAME/devops-demo:latest from Docker Hub."
```

<!-- INSTRUCTOR TODO: confirm that `railway redeploy` pulls the NEW :latest image from Docker Hub (not the previously pulled one). If it does not, switch to commit-SHA image tags and update the service image via the Railway API. -->

**What this does:**
1. Uses a CircleCI image that has Node.js, so `npx` can run the Railway CLI without installing it
2. The CLI reads your project token from `RAILWAY_TOKEN`
3. `redeploy --service` tells Railway to redeploy your devops-demo service
4. `--yes` skips the "are you sure?" prompt - there is nobody to answer it in CI
5. Railway pulls the latest image from Docker Hub and replaces the running container

**Notice what the deploy job does NOT need:**
- No `checkout` - nothing is built or uploaded, so the source code isn't needed
- No `railway login` or `railway link` - the project token is enough

**4.4: Update Workflow**

Update your workflow to include the deploy job:

```yml
workflows:
  build_test_publish_deploy:
    jobs:
      - build
      - test:
          requires:
            - build
      - publish:
          requires:
            - test
      - deploy:
          requires:
            - publish
```

**Flow:** build → test → publish → deploy

---

### Complete Updated config.yml

Here's your complete file for reference:

```yml
version: 2.1

jobs:
  build:
    docker:
      - image: cimg/openjdk:21.0
    steps:
      - checkout
      - restore_cache:
          keys:
            - maven-deps-{{ checksum "pom.xml" }}
            - maven-deps-
      - run:
          name: Install dependencies and build
          command: |
            echo "Building the application..."
            mvn clean install -DskipTests
      - save_cache:
          paths:
            - ~/.m2
          key: maven-deps-{{ checksum "pom.xml" }}
      - persist_to_workspace:
          root: .
          paths:
            - target/*.jar

  test:
    docker:
      - image: cimg/openjdk:21.0
    steps:
      - checkout
      - restore_cache:
          keys:
            - maven-deps-{{ checksum "pom.xml" }}
            - maven-deps-
      - run:
          name: Run tests
          command: |
            echo "Running tests..."
            mvn test
      - store_test_results:
          path: target/surefire-reports
      - store_artifacts:
          path: target/surefire-reports

  publish:
    docker:
      - image: cimg/base:stable
    steps:
      - checkout
      - attach_workspace:
          at: .
      - setup_remote_docker
      - run:
          name: Build Docker image
          command: |
            echo "Building Docker image..."
            docker build -t $DOCKER_USERNAME/devops-demo:latest .
      - run:
          name: Push to Docker Hub
          command: |
            echo "Logging in to Docker Hub..."
            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
            echo "Pushing image to Docker Hub..."
            docker push $DOCKER_USERNAME/devops-demo:latest

  deploy:
    docker:
      - image: cimg/node:lts
    steps:
      - run:
          name: Redeploy latest image on Railway
          command: |
            echo "Asking Railway to redeploy the latest image..."
            npx --yes @railway/cli redeploy --service "$RAILWAY_SERVICE_ID" --yes
            echo "Railway is now pulling $DOCKER_USERNAME/devops-demo:latest from Docker Hub."

workflows:
  build_test_publish_deploy:
    jobs:
      - build
      - test:
          requires:
            - build
      - publish:
          requires:
            - test
      - deploy:
          requires:
            - publish
```

---

### Step 5: Commit and Push Changes

```bash
git add .circleci/config.yml
git commit -m "Add CD: Deploy to Railway automatically"
git push origin main
```

**Watch the Magic! ✨**

1. Go to CircleCI: [https://app.circleci.com](https://app.circleci.com)
2. Your pipeline should trigger automatically
3. Watch all jobs run:
   - ⏳ build (compiling...)
   - ⏳ test (running tests...)
   - ⏳ publish (pushing to Docker Hub...)
   - ⏳ deploy (asking Railway to redeploy...)

**Timeline:**
- Build: ~2-3 minutes
- Test: ~2 minutes
- Publish: ~2-3 minutes
- Deploy: < 1 minute (one CLI call)
- **Railway deployment:** ~1 minute (pull image and start - no build)

**Total:** ~7-9 minutes from push to production!

---

### Step 6: Verify Deployment

**6.1: Check CircleCI**

All 4 jobs should be green ✅:
- build ✅
- test ✅
- publish ✅
- deploy ✅

**6.2: Check Railway**

1. Go to Railway dashboard: [https://railway.app](https://railway.app)
2. Click on your devops-demo project
3. Click on your devops-demo service
4. Click **"Deployments"** tab
5. You should see a new deployment, started just after the CircleCI deploy job
6. Open its logs - the image is pulled, not built

**6.3: Test Your Application**

Once Railway shows "Active":

```bash
curl https://devops-demo-production.up.railway.app/hello
```

**Expected response:**
```
DevOps demo application is running!
```

**Success! You have full CI/CD automation!** 🎉

---

## Part 4 - Testing the Full Pipeline

Let's make a code change to see the full automation in action!

### Step 1: Make a Code Change

**Edit your controller:**

Open `src/main/java/com/example/devopsdemo/controller/DemoController.java`

Change the message:

```java
@GetMapping("/hello")
public String hello() {
    return "DevOps demo - AUTOMATED DEPLOYMENT WORKS! 🚀";
}
```

---

### Step 2: Push the Change

```bash
git add .
git commit -m "Update hello message to test CD pipeline"
git push origin main
```

---

### Step 3: Watch the Automation

**In CircleCI:**
1. Pipeline triggers automatically
2. build → test → publish → deploy
3. All jobs complete successfully

**In Railway:**
1. Receives the redeploy request from the CircleCI deploy job
2. Pulls the new image from Docker Hub
3. Replaces the running container with the new version

**Timeline:** ~7-9 minutes total

---

### Step 4: Verify Change is Live

```bash
curl https://devops-demo-production.up.railway.app/hello
```

**You should see the NEW message:**
```
DevOps demo - AUTOMATED DEPLOYMENT WORKS! 🚀
```

**🎉 Congratulations!** You just deployed to production by pushing code!

---

## Part 5 - Understanding What You Built

### The Complete CI/CD Pipeline

**1. Code Push (You)**
```
Developer: git push origin main
↓
GitHub: Code received, webhook sent to CircleCI
```

**2. Continuous Integration (CircleCI)**
```
Build Job:
  - Clone code from GitHub
  - Compile Java code with Maven
  - Create JAR file
  - Cache dependencies for faster future builds
  
Test Job:
  - Run all unit tests
  - Ensure code quality
  - If tests fail → STOP (don't deploy broken code!)
  
Publish Job:
  - Build Docker image
  - Push to Docker Hub
  - Image ready for deployment
```

**3. Continuous Deployment (CircleCI → Railway)**
```
Deploy Job:
  - Run the Railway CLI with npx
  - Authenticate with the Railway project token
  - Ask Railway to redeploy the devops-demo service

Railway:
  - Pull the new image from Docker Hub (no build - CI already built it)
  - Stop old version
  - Start new version
  - Route traffic to new version
```

**4. Production (Railway)**
```
Users access: https://devops-demo-production.up.railway.app
They see your latest changes!
```

**Total time:** ~7-9 minutes from code push to production!

---

### Professional DevOps Practices You're Using

**1. Automated Testing** — Code is tested before deployment, broken code never reaches production

**2. Containerization** — Application runs the same everywhere, "works on my machine" problem solved

**3. Continuous Deployment** — Fast feedback loop, small frequent changes, easy to rollback

**4. Infrastructure as Code** — `.circleci/config.yml` defines your pipeline, version controlled and reproducible

**5. Environment Variables** — Credentials secure, different configs for different environments

---

## Troubleshooting Guide

### Issue 1: Deploy job fails - "Unauthorized" or "Invalid token"

**Solution:**
1. Verify `RAILWAY_TOKEN` in CircleCI environment variables (no extra spaces)
2. Make sure it is a **project token** (Project Settings → Tokens), not an account token
3. Check the token was created for the **production** environment
4. Regenerate the token in Railway if needed and update CircleCI

---

### Issue 2: "Service not found"

**Solution:**
1. Verify `RAILWAY_SERVICE_ID` in CircleCI
2. It should be a UUID (format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)
3. Copy it again from the **service** Settings (not the project settings)
4. Make sure the project token belongs to the same project as the service

---

### Issue 3: Railway can't pull the image - "not found" or "access denied"

**Solution:**
1. Check the image name in the service Settings → Source matches Docker Hub exactly: `YOUR_DOCKERHUB_USERNAME/devops-demo:latest`
2. Make sure the Docker Hub repository is **Public**
3. Confirm the CircleCI **publish** job pushed the image (check the tag `latest` on Docker Hub)

---

### Issue 4: Deploy succeeds but the old version is still running

**Solution:**
1. Check the CircleCI **publish** job finished before **deploy** (the workflow `requires: - publish`)
2. On Docker Hub, check `latest` was updated at the time of your push
3. In Railway → Deployments, check a new deployment started after the deploy job
4. Clear your browser cache, or test with `curl`

---

### Issue 5: Application deployed but /hello returns 404

**Solution:**
1. Check Railway logs: Application should show "Started Application in X seconds"
2. Verify Spring Boot is running on port 8080
3. Check port in Railway Settings → Networking is 8080

---

### Issue 6: CircleCI pipeline doesn't trigger

**Solution:**
1. Go to GitHub repo → Settings → Webhooks
2. Verify CircleCI webhook exists
3. Check branch name matches your push (main vs master)
4. Manually trigger in CircleCI to test

---

### Issue 7: Railway deploys on every push, even when tests fail

**Solution:**
Your service is connected to your GitHub repository instead of the Docker image.
1. Open the service → **Settings** → **Source**
2. Disconnect the GitHub repository
3. Set the source to your Docker image: `YOUR_DOCKERHUB_USERNAME/devops-demo:latest`

Only CircleCI should trigger deployments.

---

## Summary

### What You Accomplished Today

1. ✅ Deployed your Docker Hub image to Railway
2. ✅ Created complete CI/CD pipeline (build → test → publish → deploy)
3. ✅ Automated deployment with a Railway project token and `railway redeploy`
4. ✅ Tested full pipeline with code changes
5. ✅ Learned professional DevOps practices

---

## Additional Resources

- [What is Continuous Deployment? — Atlassian](https://www.atlassian.com/continuous-delivery/continuous-deployment)
- [CD Best Practices — Google Cloud](https://cloud.google.com/architecture/devops/devops-tech-continuous-delivery)
- [CI/CD Explained — Red Hat](https://www.redhat.com/en/topics/devops/what-is-ci-cd)
- [Railway Docs](https://docs.railway.app/)
- [Deploying with the Railway CLI](https://docs.railway.com/cli/deploying)
- [Deployments Guide — Railway](https://docs.railway.app/deploy/deployments)
- [CircleCI Deployment Overview](https://circleci.com/docs/deployment-overview/)
- [CircleCI Environment Variables](https://circleci.com/docs/env-vars/)
- [Railway Status](https://status.railway.app/)
- [CircleCI Status](https://status.circleci.com/)