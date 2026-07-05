---
marp: true
theme: default
paginate: true
header: 'Jenkins — Practical Examples'
footer: 'Simple guide for everyone'
style: |
  section { font-size: 26px; }
  h1 { color: #2563eb; }
  h2 { color: #1e40af; }
  code { background: #f1f5f9; font-size: 22px; }
  table { font-size: 22px; }
---

# Jenkins
## Practical Examples (Beginner Friendly)

**From zero to automated builds and deployments — with real examples you can run**

---

# Who Is This For?

This guide is for you if:

- You heard about **Jenkins** but don't know what it does
- You build and deploy software manually and want to **automate** it
- You want **simple explanations** and **real pipeline examples**

**You do NOT need to be a DevOps expert to follow this guide.**

---

# A Simple Story First

Imagine you built a **web shop** app.

Every time a developer pushes code, someone must:

1. Pull the latest code
2. Run tests
3. Build a Docker image
4. Push image to a registry
5. Deploy to Test server

**If you do this by hand, every day, mistakes happen.**

Jenkins automates these steps for you.

---

# What Is CI/CD? (Before Jenkins)

| Term | Simple meaning | Example |
|------|----------------|---------|
| **CI** (Continuous Integration) | Automatically build & test every code change | Run unit tests on every Git push |
| **CD** (Continuous Delivery) | Automatically prepare release artifacts | Build Docker image, store in registry |
| **CD** (Continuous Deployment) | Automatically deploy to servers | Deploy to Test after tests pass |

**Jenkins is the robot that runs CI/CD pipelines.**

---

# The Problem (Without Jenkins)

**Manual release process:**

```
Developer pushes code
    → someone runs tests locally (maybe)
    → someone builds on their laptop
    → someone copies JAR/Docker image to server
    → someone restarts the app
    → nobody remembers exact steps next week
```

**Result:**
- "Works on my machine" bugs
- Slow releases
- No audit trail
- Fear of deploying on Friday

---

# What Is Jenkins? (1 minute)

**Jenkins is an open-source automation server.**

It watches your Git repo and runs **pipelines** (scripts) when events happen:

- Code pushed to `main`
- Pull request opened
- Nightly schedule (cron)
- Manual button click

**One job:** turn human release steps into repeatable, logged automation.

---

# Jenkins in One Picture

```
  Developer                Jenkins                    Target
  ─────────                ───────                    ──────
  git push  ──────────►  Pipeline runs  ──────────►  Test server
                         1. checkout code            Staging
                         2. run tests                Production
                         3. build image
                         4. push to registry
                         5. deploy
```

**Jenkins is the middle layer between Git and your environments.**

---

# 5 Words You Must Know

| Word | Simple meaning | Example |
|------|----------------|---------|
| **Job / Pipeline** | Automated workflow | `build-my-shop` |
| **Stage** | One logical step in a pipeline | `Test`, `Build`, `Deploy` |
| **Agent / Node** | Machine that runs the work | Jenkins server or worker |
| **Plugin** | Extension that adds features | Docker, Git, Kubernetes |
| **Credential** | Secret stored safely in Jenkins | Git password, API token |

---

# Jenkins Architecture (Simple View)

```
┌─────────────────────────────────────────────┐
│              Jenkins Controller              │
│  (UI, scheduling, stores config & secrets)   │
└──────────────┬──────────────┬───────────────┘
               │              │
        ┌──────▼──────┐ ┌─────▼──────┐
        │  Agent 1    │ │  Agent 2   │
        │  (Linux)    │ │  (Docker)  │
        └─────────────┘ └────────────┘
```

- **Controller** = brain (web UI, job definitions)
- **Agent** = worker (runs build commands)
- Heavy builds run on agents, not on the controller

---

# Install Jenkins (Quick Start)

**Docker (easiest for learning):**
```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

**Get initial admin password:**
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

**Open:** `http://localhost:8080` → install suggested plugins → create admin user.

---

# First Login Walkthrough

After install you see:

1. **Dashboard** — list of all jobs/pipelines
2. **New Item** — create a job
3. **Manage Jenkins** — plugins, credentials, nodes
4. **Build History** — every run with logs

**Think of Dashboard as your automation control room.**

---

# Example 1: Freestyle Job (Simplest)

**Goal:** Print "Hello" when you click Build.

1. **New Item** → name: `hello-world` → type: **Freestyle project**
2. **Build Steps** → **Execute shell:**
```bash
echo "Hello from Jenkins!"
date
whoami
```
3. Click **Build Now**
4. Open build `#1` → **Console Output**

**You just ran your first automated task.**

---

# Example 2: Freestyle Job — Build a Java App

**Build Steps → Execute shell:**
```bash
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
./mvnw clean package -DskipTests
ls -la target/*.jar
```

**Post-build:** archive `target/*.jar` as artifact.

**Result:** Every build produces a downloadable JAR file in Jenkins UI.

---

# Example 3: Pipeline Job (Modern Way)

**Freestyle** = click UI, limited reuse.
**Pipeline** = code in a `Jenkinsfile`, versioned in Git.

1. **New Item** → `my-pipeline` → type: **Pipeline**
2. Pipeline script:

```groovy
pipeline {
    agent any
    stages {
        stage('Hello') {
            steps {
                echo 'Hello from Pipeline!'
            }
        }
    }
}
```

3. **Build Now** → check Console Output

---

# Declarative vs Scripted Pipeline

| Type | Style | Best for |
|------|-------|----------|
| **Declarative** | Structured `pipeline { }` block | Most teams (recommended) |
| **Scripted** | Free-form Groovy | Advanced custom logic |

**This guide focuses on Declarative** — easier to read and maintain.

---

# Example 4: Real Pipeline — Test and Build

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/my-team/my-shop.git'
            }
        }
        stage('Test') {
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
```

**Stages run in order.** If Test fails, Build never runs.

---

# Example 5: Jenkinsfile in Git (Best Practice)

Store pipeline **inside your app repo**:

```
my-shop/
├── src/
├── Dockerfile
├── package.json
└── Jenkinsfile        ← pipeline lives here
```

**Jenkins job config:**
- Definition: **Pipeline script from SCM**
- SCM: Git
- Script path: `Jenkinsfile`

**Now pipeline changes go through code review like app code.**

---

# Example 6: Full Jenkinsfile for a Web App

```groovy
pipeline {
    agent any

    environment {
        APP_NAME = 'my-shop'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Test') {
            steps {
                sh 'npm ci'
                sh 'npm test'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${APP_NAME}:${IMAGE_TAG} ."
            }
        }
        stage('Push Image') {
            steps {
                sh "docker push registry.example.com/${APP_NAME}:${IMAGE_TAG}"
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed — check logs.'
        }
    }
}
```

---

# Example 7: Stages Explained

```groovy
stage('Test') {
    steps {
        sh 'npm test'
    }
}
```

| Part | Meaning |
|------|---------|
| `stage('Test')` | Named step shown in Blue Ocean / UI |
| `steps { }` | Commands to run |
| `sh '...'` | Run shell command on agent |
| `echo '...'` | Print message to log |

**Failed step = pipeline stops** (unless you add `try/catch` or `catchError`).

---

# Example 8: Parallel Stages (Faster Builds)

Run independent checks at the same time:

```groovy
stage('Quality Checks') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'npm test' }
        }
        stage('Lint') {
            steps { sh 'npm run lint' }
        }
        stage('Security Scan') {
            steps { sh 'npm audit' }
        }
    }
}
```

**All three run together** → pipeline finishes faster.

---

# Example 9: Deploy to Test Environment

```groovy
stage('Deploy to Test') {
    when {
        branch 'main'
    }
    steps {
        sh '''
          kubectl set image deployment/my-shop \
            my-shop=registry.example.com/my-shop:${BUILD_NUMBER} \
            -n shop-test
        '''
    }
}
```

**`when { branch 'main' }`** = only deploy from main branch, not feature branches.

---

# Example 10: Multi-Environment Pipeline

```groovy
stage('Deploy') {
    steps {
        script {
            if (env.BRANCH_NAME == 'develop') {
                sh './deploy.sh test'
            } else if (env.BRANCH_NAME == 'main') {
                input message: 'Deploy to Production?', ok: 'Deploy'
                sh './deploy.sh prod'
            }
        }
    }
}
```

| Branch | Action |
|--------|--------|
| `feature/*` | Test + Build only |
| `develop` | Auto-deploy to Test |
| `main` | Manual approval → Production |

---

# Example 11: Credentials (Never Hardcode Secrets)

**Bad:**
```groovy
sh 'docker login -u admin -p MyPassword123 registry.example.com'
```

**Good — use Jenkins Credentials:**
```groovy
stage('Push Image') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'docker-registry',
            usernameVariable: 'USER',
            passwordVariable: 'PASS'
        )]) {
            sh 'echo $PASS | docker login -u $USER --password-stdin registry.example.com'
            sh 'docker push registry.example.com/my-shop:latest'
        }
    }
}
```

Add credential in: **Manage Jenkins → Credentials → Add**

---

# Example 12: Git Webhook (Auto-Build on Push)

**Without webhook:** Jenkins polls Git every few minutes (slow).

**With webhook:** Git notifies Jenkins instantly on push.

**Setup:**
1. Jenkins job → **Build Triggers** → **GitHub hook trigger**
2. GitHub repo → **Settings → Webhooks** → URL:
   ```
   http://your-jenkins-server:8080/github-webhook/
   ```

**Now every `git push` starts the pipeline automatically.**

---

# Example 13: Parameters (Manual Choices)

```groovy
pipeline {
    agent any

    parameters {
        choice(name: 'ENV', choices: ['test', 'staging', 'prod'], description: 'Target environment')
        string(name: 'VERSION', defaultValue: 'latest', description: 'Image tag')
    }

    stages {
        stage('Deploy') {
            steps {
                echo "Deploying version ${params.VERSION} to ${params.ENV}"
                sh "./deploy.sh ${params.ENV} ${params.VERSION}"
            }
        }
    }
}
```

**Build with Parameters** button appears in UI — useful for on-demand releases.

---

# Example 14: Docker Agent (Clean Build Every Time)

Instead of installing tools on the Jenkins server, use a fresh container:

```groovy
pipeline {
    agent {
        docker {
            image 'node:20-alpine'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'node --version'
                sh 'npm ci && npm run build'
            }
        }
    }
}
```

**Each build starts in a clean `node:20` container** — no leftover files from previous builds.

---

# Example 15: Kubernetes Agent (Cloud Builds)

Jenkins spawns a Pod per build in Kubernetes:

```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
              apiVersion: v1
              kind: Pod
              spec:
                containers:
                - name: maven
                  image: maven:3.9-eclipse-temurin-17
                  command: ["sleep"]
                  args: ["99999"]
            '''
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
    }
}
```

**Scales automatically** — many builds = many pods, then they disappear.

---

# Example 16: Post Actions (Always Run Cleanup)

```groovy
post {
    always {
        echo 'Pipeline finished.'
        cleanWs()    // clean workspace
    }
    success {
        slackSend channel: '#releases', message: "Build ${env.BUILD_NUMBER} succeeded"
    }
    failure {
        emailext subject: "Build Failed: ${env.JOB_NAME}",
                 body: "Check ${env.BUILD_URL}",
                 to: 'team@example.com'
    }
}
```

| Block | When it runs |
|-------|--------------|
| `always` | Every time, pass or fail |
| `success` | Only if all stages passed |
| `failure` | Only if something failed |
| `aborted` | User cancelled the build |

---

# Example 17: Shared Library (Reuse Across Projects)

Put common functions in a separate Git repo:

**`vars/deployApp.groovy`:**
```groovy
def call(String env, String tag) {
    sh "./scripts/deploy.sh ${env} ${tag}"
}
```

**In any Jenkinsfile:**
```groovy
@Library('my-shared-library') _
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                deployApp('test', "${env.BUILD_NUMBER}")
            }
        }
    }
}
```

**Write once, use in 50 projects.**

---

# Example 18: Multibranch Pipeline

One Jenkins job automatically discovers **all branches**:

```
my-shop/
├── main           → pipeline runs → deploy to prod (with approval)
├── develop        → pipeline runs → deploy to test
└── feature/login  → pipeline runs → test only
```

**Setup:** New Item → **Multibranch Pipeline** → point to Git repo.

Jenkins creates a sub-job per branch automatically.

---

# Jenkins + Docker + Kubernetes Flow

Full modern pipeline for a containerized app:

```
git push
  → Jenkins: checkout + test
  → Jenkins: docker build
  → Jenkins: push to registry (ghcr.io / ECR / Harbor)
  → Jenkins: update image tag in GitOps repo (or trigger Argo CD)
  → Argo CD / Kargo: deploy to cluster
```

**Jenkins builds. GitOps tools deploy.** Clean separation.

---

# Real Scenario: Day in the Life

**08:45** — Developer pushes `feature/new-checkout` to Git
**08:46** — Webhook triggers Jenkins Multibranch Pipeline
**08:50** — Unit tests + lint pass ✅ (Build #142)
**10:00** — PR merged to `develop`
**10:01** — Jenkins deploys `my-shop:142` to Test
**14:00** — QA approves
**14:30** — Release manager merges `develop` → `main`
**14:31** — Jenkins builds #143, waits for manual approval
**14:35** — Manager clicks **Deploy** → Production updated
**14:40** — Slack notification: "Release 2.3.0 live" ✅

---

# Jenkins UI Tools

| Tool | What it does |
|------|--------------|
| **Classic UI** | Traditional job list and config forms |
| **Blue Ocean** | Modern pipeline visualization (stages as graph) |
| **Console Output** | Full log of every build step |
| **Build History** | Timeline of all runs with status icons |

**Install Blue Ocean:** Manage Jenkins → Plugins → search `Blue Ocean`

---

# Useful Plugins to Start With

| Plugin | Purpose |
|--------|---------|
| **Git / GitHub** | Pull code, webhooks |
| **Pipeline** | Jenkinsfile support |
| **Docker Pipeline** | Build inside Docker |
| **Kubernetes** | Run agents as K8s pods |
| **Credentials Binding** | Inject secrets safely |
| **JUnit / Allure** | Test report dashboards |
| **Slack / Email** | Notifications |

**Don't install everything** — add plugins when you need them.

---

# Common Mistakes (And Fixes)

| Mistake | Fix |
|---------|-----|
| Running builds on controller | Use agents for all build work |
| Passwords in Jenkinsfile | Use Jenkins Credentials |
| No `Jenkinsfile` in Git | Store pipeline as code |
| Giant one-stage pipeline | Split into clear stages |
| No post cleanup | Add `post { always { cleanWs() } }` |
| Deploy on every branch | Use `when { branch 'main' }` |
| Too many plugins | Install only what you need |
| No build history retention | Set "Discard old builds" in job config |

---

# Jenkins vs Other CI Tools (Quick View)

| | **Jenkins** | **GitHub Actions** | **GitLab CI** |
|---|-------------|-------------------|---------------|
| Hosting | Self-hosted (you manage server) | Cloud (GitHub) | Cloud or self-hosted |
| Config | Jenkinsfile (Groovy) | YAML in `.github/workflows/` | `.gitlab-ci.yml` |
| Plugins | Huge ecosystem | GitHub marketplace | Built-in integrations |
| Best for | On-prem, custom workflows | GitHub-hosted projects | GitLab users |

**Jenkins shines when you need full control on your own infrastructure.**

---

# Security Basics

1. **Never expose Jenkins to public internet** without authentication
2. **Use RBAC** — developers can build, only admins configure
3. **Store secrets in Credentials**, not in jobs or Git
4. **Keep Jenkins LTS updated** — security patches matter
5. **Audit plugins** — remove unused ones
6. **Use HTTPS** for the web UI

---

# Jenkins Command Cheat Sheet

```groovy
// Pipeline skeleton
pipeline {
    agent any
    environment { KEY = 'value' }
    parameters { string(name: 'VER', defaultValue: '1.0') }
    stages {
        stage('Build') {
            when { branch 'main' }
            steps { sh 'make build' }
        }
    }
    post {
        always  { cleanWs() }
        success { echo 'OK' }
        failure { echo 'FAILED' }
    }
}
```

**Shell steps:** `sh 'command'` (Linux) · `bat 'command'` (Windows)
**Git checkout:** `checkout scm`
**Input approval:** `input message: 'Approve?'`
**Parallel:** `parallel { stage('A'){...} stage('B'){...} }`

---

# Try It Yourself (Mini Lab)

**Time: ~30 minutes**

```bash
# 1. Start Jenkins
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts

# 2. Get password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# 3. Open http://localhost:8080 — complete setup wizard

# 4. Create Pipeline job with this Jenkinsfile:
```

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/spring-projects/spring-petclinic.git'
            }
        }
        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
}
```

```bash
# 5. Build Now → download JAR from build page
# 6. Add a failing test stage, observe pipeline stops
# 7. Add post { failure { echo 'fix me' } }
# 8. Cleanup: docker rm -f jenkins && docker volume rm jenkins_home
```

---

# Practice Questions

1. What is the difference between a **Freestyle job** and a **Pipeline**?
2. Where should the `Jenkinsfile` live in a real project?
3. How do you prevent passwords from appearing in logs?
4. What does `when { branch 'main' }` do?
5. Why use agents instead of running builds on the controller?

---

# Practice Answers

1. **Freestyle** = configured in UI, hard to version. **Pipeline** = code in `Jenkinsfile`, reviewable in Git.
2. In the **root of the application repository**, committed alongside source code.
3. Store secrets in **Jenkins Credentials** and inject with `withCredentials` — never hardcode.
4. Runs that stage **only when the branch is `main`** — skips deploy on feature branches.
5. Agents isolate builds, provide scalable workers, and keep the controller stable and secure.

---

# Final Summary

If you remember only 4 things:

1. **Jenkins automates** build, test, and deploy steps
2. **Pipeline as Code** (`Jenkinsfile` in Git) is the modern standard
3. **Stages** make pipelines readable; **agents** run the work safely
4. **Jenkins builds artifacts** — pair it with GitOps (Argo CD, Kargo) for deployments

---

# How Jenkins Fits the Bigger Picture

```
Developer → Git push
         → Jenkins (CI): test, build image, push registry
         → GitOps repo updated (image tag)
         → Argo CD / Kargo (CD): deploy to Kubernetes
```

| Tool | Role |
|------|------|
| **Jenkins** | Continuous Integration — build & test |
| **Helm** | Package Kubernetes apps |
| **Argo CD** | Deploy from Git to cluster |
| **Kargo** | Promote between environments |

---

# Next Steps

- **Beginner:** complete the Mini Lab with Docker Jenkins
- **Intermediate:** move a Freestyle job to `Jenkinsfile` in Git
- **Advanced:** Kubernetes agents + shared library + GitOps deploy stage

**You are ready to explain Jenkins to your team.**

---

# Thank You

**Official docs:**
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Jenkinsfile Best Practices](https://www.jenkins.io/doc/book/pipeline/pipeline-best-practices/)

Questions? Run the Mini Lab — building one real pipeline teaches more than reading ten slides.
