Great. Here's a **step-by-step guide** for completing your GitHub Actions CI/CD pipeline project with **deployment to AWS** using a Node.js app. We’ll break it into actionable phases, from repo setup to deployment and release.

---

## ✅ CI/CD Pipeline Project with GitHub Actions (Deploy to AWS)

---

### **PHASE 1: Project & Environment Setup**

#### 1.1 Create a GitHub Repository

* Go to GitHub and create a new repo (e.g., `ci-cd-aws-deploy`)
* Clone the repo to your local machine:

  ```bash
  git clone https://github.com/your-username/ci-cd-aws-deploy.git
  cd ci-cd-aws-deploy
  ```

#### 1.2 Initialize Node.js App

```bash
npm init -y
echo 'console.log("App deployed via GitHub Actions")' > app.js
```

Push the app:

```bash
git add .
git commit -m "Initial commit"
git push origin main
```

#### 1.3 Setup AWS CLI & IAM

* Create an IAM user with programmatic access
* Attach `AmazonS3FullAccess` or appropriate policy (for S3 deployment)
* Save `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`

#### 1.4 Add GitHub Secrets

In your GitHub repo:

* Go to **Settings → Secrets and variables → Actions**
* Add:

  * `AWS_ACCESS_KEY_ID`
  * `AWS_SECRET_ACCESS_KEY`

---

### **PHASE 2: CI/CD Workflows**

#### 2.1 Create `.github/workflows/ci.yml`

Runs on every push and installs dependencies:

```yaml
name: CI

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: echo "No tests yet"
```

Commit & push:

```bash
git add .github
git commit -m "Add CI workflow"
git push
```

Screenshot placeholder:
`![github](img/ci-pipeline.png)`

---

#### 2.2 Create `.github/workflows/bump-version.yml`

Auto-increment version on main branch push:

```yaml
name: Bump version and tag

on:
  push:
    branches:
      - main

jobs:
  tag:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Bump version
        uses: anothrNick/github-tag-action@1.67.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_prefix: "v"
          default_bump: patch
```

Screenshot placeholder:
`![github](img/version-tag.png)`

---

#### 2.3 Create `.github/workflows/deploy-to-aws.yml`

Deploy your code to an S3 bucket.

```yaml
name: Deploy to AWS S3

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to S3
        run: |
          aws s3 mb s3://your-unique-bucket-name --region us-east-1 || true
          aws s3 cp ./ s3://your-unique-bucket-name/ --recursive --exclude ".git/*" --exclude "node_modules/*"
```

> Replace `your-unique-bucket-name` with your actual S3 bucket name.

Screenshot placeholder:
`![github](img/aws-deploy-success.png)`

---

#### 2.4 Create `.github/workflows/create-release.yml`

Create a GitHub Release when a tag is pushed.

```yaml
name: Create Release

on:
  push:
    tags:
      - '*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
```

Screenshot placeholder:
`![github](img/release-page.png)`

---

### **PHASE 3: Final Steps & Validation**

#### ✅ Test Everything

1. Push to `main` → triggers CI → bumps version → deploys to AWS S3
2. Push a tag `v1.0.0` → triggers GitHub Release
3. Confirm:

   * CI runs successfully
   * Deployment uploads to S3
   * Version is tagged
   * Release appears on GitHub

#### 📁 Folder Recap

```
.
├── .github/workflows/
│   ├── ci.yml
│   ├── bump-version.yml
│   ├── deploy-to-aws.yml
│   └── create-release.yml
├── app.js
├── package.json
├── README.md
└── img/
```
