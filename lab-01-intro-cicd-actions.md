# Lab 1: Introduction to CI/CD and GitHub Actions

**Duration:** 45-60 minutes  
**Level:** Beginner  
**Prerequisites:** Basic Git knowledge, GitHub account

---

## Overview

This lab introduces you to Continuous Integration/Continuous Deployment (CI/CD) concepts and GitHub Actions. Through hands-on exercises, you'll explore how modern software teams automate their workflows and understand the core components that make GitHub Actions powerful.

All exercises can be performed directly in your web browser using GitHub's interface.

---

## Exercise 1: Understanding CI/CD Through Examples (10 minutes)

### Explore a real-world CI/CD pipeline

1. **Visit a popular open-source repository**:
   - Go to: https://github.com/actions/toolkit
   
2. **Click the "Actions" tab** at the top

3. **Observe the workflow runs**:
   - Green checkmarks (✓) = tests passed
   - Red X marks (✗) = tests failed
   - Yellow dots (●) = currently running
   - Each run shows which commit triggered it

4. **Click on any completed workflow run** (green checkmark)

5. **Explore the execution details**:
   - See multiple jobs (boxes in the diagram)
   - Click on a job name to see steps
   - Expand steps to view detailed logs
   - Notice execution times

6. **Navigate to the workflow file**:
   - Click "Code" tab
   - Navigate to `.github/workflows/`
   - Open any `.yml` file
   - Observe the YAML structure

### What just happened?

You just witnessed a **CI/CD pipeline** in action:
- **Continuous Integration**: Code changes automatically trigger tests
- **Automated Validation**: Every commit is verified before merging
- **Immediate Feedback**: Developers know instantly if their code works
- **Quality Gates**: Failed tests prevent bad code from reaching production

This is what we'll be building throughout this course!

---

## Exercise 2: Dissecting Workflow Anatomy (10 minutes)

### Understand the structure of a workflow file

**Examine this complete workflow:**

```yaml
name: Hello World CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  greet:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Say hello
        run: echo "Hello, GitHub Actions!"
      
      - name: Show date
        run: date
      
      - name: List files
        run: ls -la
```

### Component Breakdown

| Line | Component | Purpose | Details |
|------|-----------|---------|---------|
| 1 | `name:` | Workflow name | Displayed in Actions tab |
| 3-6 | `on:` | Event triggers | When workflow runs |
| 4 | `push:` | Push event | Triggers on code push |
| 5 | `branches:` | Branch filter | Only main branch |
| 6 | `pull_request:` | PR event | Triggers on pull requests |
| 8 | `jobs:` | Jobs section | Defines work to do |
| 9 | `greet:` | Job name | Custom identifier |
| 10 | `runs-on:` | Runner OS | ubuntu-latest |
| 12 | `steps:` | Steps list | Sequential tasks |
| 13-14 | `name:` + `uses:` | Action step | Use marketplace action |
| 16-17 | `name:` + `run:` | Command step | Execute shell command |

### Visual Flow

```
Event (push/PR) 
    ↓
Workflow Triggers
    ↓
Job "greet" starts on ubuntu-latest
    ↓
Step 1: Checkout code
Step 2: Say hello
Step 3: Show date
Step 4: List files
    ↓
Job Completes (Success/Failure)
```

---

## Exercise 3: GitHub Actions Architecture Deep Dive (10 minutes)

### Understand the six core components

**Create a mental model by mapping components to real-world scenarios:**

#### 1. **Workflows** = Recipe
- YAML files in `.github/workflows/`
- Complete set of instructions
- Can have multiple workflows per repository

**Example**: `ci.yml`, `deploy.yml`, `release.yml`

#### 2. **Events** = Triggers
- Activities that start workflows
- Can be GitHub events or external

**Common events**:
```yaml
on:
  push:              # Code pushed
  pull_request:      # PR opened/updated
  schedule:          # Time-based (cron)
  workflow_dispatch: # Manual trigger
  release:           # Release published
  issues:            # Issue opened/closed
```

#### 3. **Jobs** = Tasks
- Group of steps running together
- Run on same machine (runner)
- Parallel by default, can be sequential

**Example**:
```yaml
jobs:
  build:    # Job 1
  test:     # Job 2 (runs in parallel with build)
  deploy:   # Job 3 (can wait for others)
```

#### 4. **Steps** = Instructions
- Individual actions or commands
- Execute in order
- Share the same environment

**Example**:
```yaml
steps:
  - name: Step 1
    run: echo "First"
  - name: Step 2
    run: echo "Second"
```

#### 5. **Actions** = Tools
- Reusable units of code
- From marketplace or custom
- Simplify complex operations

**Example**:
```yaml
- uses: actions/checkout@v4      # Official action
- uses: docker/build-push-action@v5  # Community action
```

#### 6. **Runners** = Computers
- Virtual machines that execute workflows
- GitHub-hosted or self-hosted
- Different OS options

**Available runners**:
- `ubuntu-latest` (Linux)
- `windows-latest` (Windows)
- `macos-latest` (macOS)

### How Components Work Together

```
┌─────────────────────────────────────────┐
│         YOUR REPOSITORY                 │
│  .github/workflows/ci.yml (Workflow)    │
└─────────────────┬───────────────────────┘
                  │
      Event: push to main branch
                  │
                  ▼
┌─────────────────────────────────────────┐
│       GITHUB ACTIONS PLATFORM           │
│  - Schedules jobs                       │
│  - Assigns runners                      │
│  - Manages execution                    │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│    RUNNER (ubuntu-latest)               │
│                                         │
│  Job: test                              │
│    Step 1: actions/checkout@v4          │
│    Step 2: run npm install              │
│    Step 3: run npm test                 │
└─────────────────────────────────────────┘
```

---

## Exercise 4: Mapping CI/CD to Real Workflows (10 minutes)

### See how teams use GitHub Actions

**Common workflow patterns:**

#### Pattern 1: Pull Request Validation
```yaml
name: PR Validation

on:
  pull_request:
    branches: [ main ]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run linter
        run: npm run lint
      - name: Run tests
        run: npm test
      - name: Check security
        run: npm audit
```

**Use case**: Ensure code quality before merging

#### Pattern 2: Continuous Deployment
```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build application
        run: npm run build
      - name: Deploy to server
        run: ./deploy.sh
```

**Use case**: Automatically deploy after merge

#### Pattern 3: Scheduled Maintenance
```yaml
name: Nightly Database Backup

on:
  schedule:
    - cron: '0 2 * * *'  # 2 AM daily

jobs:
  backup:
    runs-on: ubuntu-latest
    steps:
      - name: Backup database
        run: ./backup-script.sh
      - name: Upload to storage
        run: ./upload-backup.sh
```

**Use case**: Automated maintenance tasks

#### Pattern 4: Multi-Environment Deployment
```yaml
name: Multi-Stage Deployment

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [dev, staging, production]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to ${{ github.event.inputs.environment }}
        run: ./deploy.sh ${{ github.event.inputs.environment }}
```

**Use case**: Manual deployment to specific environments

---

## Exercise 5: Runners and Execution Environments (10 minutes)

### Understand where your code runs

**GitHub-hosted runners provide:**
- Fresh virtual environment for each job
- Pre-installed software and tools
- Automatic cleanup after execution
- No maintenance required

#### Runner Specifications

| Runner | OS | vCPUs | RAM | Storage |
|--------|----|----|-----|---------|
| `ubuntu-latest` | Ubuntu 22.04 | 2 | 7 GB | 14 GB SSD |
| `windows-latest` | Windows Server 2022 | 2 | 7 GB | 14 GB SSD |
| `macos-latest` | macOS 14 | 3 | 14 GB | 14 GB SSD |

#### What's Pre-installed?

**ubuntu-latest includes**:
- Languages: Node.js, Python, Java, Ruby, Go, PHP, .NET
- Tools: Git, Docker, kubectl, Terraform, AWS CLI
- Build tools: Make, CMake, Gradle, Maven
- Databases: PostgreSQL, MySQL (for testing)

**View full list**: https://github.com/actions/runner-images

#### Choosing the Right Runner

```yaml
jobs:
  linux-build:
    runs-on: ubuntu-latest    # Most common, fastest
  
  windows-build:
    runs-on: windows-latest   # For Windows-specific builds
  
  mac-build:
    runs-on: macos-latest     # For iOS/macOS apps
```

**Cost**: Free for public repos, included minutes for private repos

---

## Exercise 6: GitHub Actions vs Other CI/CD Tools (10 minutes)

### See how GitHub Actions compares

| Feature | GitHub Actions | Jenkins | GitLab CI | CircleCI |
|---------|---------------|---------|-----------|----------|
| **Hosting** | GitHub-hosted | Self-hosted | GitLab-hosted | Cloud |
| **Configuration** | YAML in repo | Jenkinsfile | `.gitlab-ci.yml` | `config.yml` |
| **Integration** | Native GitHub | Plugins | Native GitLab | Via API |
| **Marketplace** | Yes (thousands) | Plugins | Limited | Orbs |
| **Free tier** | 2,000 min/month | Unlimited (self) | 400 min/month | 6,000 min/month |
| **Setup effort** | Minimal | High | Minimal | Minimal |
| **Matrix builds** | Yes | Yes | Yes | Yes |
| **Self-hosted** | Optional | Required | Optional | No |

### Why GitHub Actions?

**Advantages**:
✓ Deeply integrated with GitHub (events, permissions, API)
✓ No separate platform to manage
✓ Workflow files live with code
✓ Rich marketplace of actions
✓ Easy to get started
✓ Generous free tier

**Best for**:
- Projects already on GitHub
- Teams wanting simple CI/CD
- Open source projects (unlimited free)
- Rapid prototyping

---

## Exercise 7: Hands-On Analysis Challenge (10 minutes)

### Analyze this complete workflow

```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
    paths-ignore:
      - '**.md'
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/
  
  build:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build application
        run: npm run build
      
      - name: Upload build
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
  
  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    
    steps:
      - name: Deploy to production
        run: echo "Deploying..."
```

**Answer these questions:**

1. **How many jobs are in this workflow?**
2. **Do they run in parallel or sequence?**
3. **When does this workflow trigger?**
4. **What happens on develop branch push?**
5. **Which job only runs on main branch?**
6. **What marketplace actions are used?**
7. **What artifacts are created?**

### Answers

1. **Three jobs**: test, build, deploy
2. **Sequential**: test → build → deploy (due to `needs:` dependencies)
3. **Triggers on**: 
   - Push to main or develop (except .md files)
   - Pull requests targeting main
4. **Develop branch**: test and build run, deploy is skipped (only runs on main)
5. **Deploy job**: Only runs on main (`if: github.ref == 'refs/heads/main'`)
6. **Marketplace actions**:
   - `actions/checkout@v4`
   - `actions/setup-node@v4`
   - `actions/upload-artifact@v4`
7. **Artifacts created**:
   - `coverage` (test coverage reports)
   - `build` (compiled application)

---

## What You've Learned

Through these exercises, you've mastered the fundamentals:

### 1. **CI/CD Concepts** (Exercise 1)
- Automated testing and deployment
- Immediate feedback on code changes
- Quality gates prevent bad code

### 2. **Workflow Structure** (Exercise 2)
- YAML syntax and organization
- Event triggers and job definitions
- Steps with actions and commands

### 3. **Core Components** (Exercise 3)
- Workflows, events, jobs, steps, actions, runners
- How components interact
- Architecture and execution flow

### 4. **Real-World Patterns** (Exercise 4)
- PR validation workflows
- Continuous deployment
- Scheduled tasks
- Multi-environment deployments

### 5. **Runner Environments** (Exercise 5)
- GitHub-hosted runners
- Pre-installed software
- OS options and specifications

### 6. **Tool Comparison** (Exercise 6)
- GitHub Actions vs competitors
- Advantages and use cases
- Integration benefits

### 7. **Practical Analysis** (Exercise 7)
- Reading complex workflows
- Understanding dependencies
- Identifying conditional execution

---

## Best Practices

**DO:**
- Start with simple workflows and build complexity
- Use descriptive names for workflows, jobs, and steps
- Leverage marketplace actions (don't reinvent the wheel)
- Keep workflows focused (separate concerns)
- Add comments for complex logic
- Use `workflow_dispatch` for testing

**DON'T:**
- Put secrets directly in workflow files
- Create overly complex workflows
- Ignore failed workflow runs
- Use deprecated actions
- Run unnecessary jobs (waste minutes)

---

## CI/CD Benefits Summary

### Before CI/CD
❌ Manual testing before each release  
❌ Integration problems discovered late  
❌ Inconsistent deployment process  
❌ Delayed feedback on code quality  
❌ Fear of releasing often  

### After CI/CD
✅ Automated testing on every commit  
✅ Early detection of integration issues  
✅ Repeatable, automated deployments  
✅ Immediate feedback to developers  
✅ Confidence to release frequently  

---

## Glossary

| Term | Definition |
|------|------------|
| **Workflow** | Automated process defined in YAML file |
| **Event** | Activity that triggers a workflow |
| **Job** | Set of steps that run on same runner |
| **Step** | Individual task (action or command) |
| **Action** | Reusable unit of code |
| **Runner** | Server that executes workflows |
| **Artifact** | File or set of files produced by workflow |
| **Secret** | Encrypted environment variable |
| **Matrix** | Strategy to run job with multiple configurations |

---

## Next Steps

Now that you understand CI/CD and GitHub Actions fundamentals, you're ready to create your first workflow:
👉 [Lab 2: Your First Workflow](./lab-02-first-workflow.md)

---

**End of Lab 1 - Introduction to CI/CD and GitHub Actions**
