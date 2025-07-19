Certainly! Here's the **updated AWS Configuration Cheat Sheet**, fully integrated with your **GitHub Actions deployment workflow** instead of the original Travis CI section. It’s formatted in Markdown and structured for clarity and readability.

---

# 📝 AWS Configuration Cheat Sheet (GitHub Actions Version)

> This guide provides a concise reference for setting up AWS infrastructure and automating deployments using GitHub Actions. It supplements video material and helps you verify each step—especially when AWS UI changes.

---

## 🔄 Update Docker Compose Configuration

* Rename the development Docker Compose file:

  ```bash
  mv docker-compose.yml docker-compose-dev.yml
  ```

---

## 🛡️ Create EC2 IAM Instance Profile

1. Open the **AWS Management Console**.
2. Navigate to **IAM**.
3. In the sidebar, select **Roles** → **Create role**.
4. Choose **AWS service** as the trusted entity, then select **EC2**.
5. Attach these policies:

   * `AWSElasticBeanstalkWebTier`
   * `AWSElasticBeanstalkWorkerTier`
   * `AWSElasticBeanstalkMulticontainerDocker`
6. Name the role:
   `aws-elasticbeanstalk-ec2-role`
7. Click **Create role**.

---

## 🌱 Create Elastic Beanstalk Environment

1. In the AWS Console, search for **Elastic Beanstalk**.
2. Click **Create Application** (or **Create Environment** if you’ve used EB before).
3. Provide an **Application name** (this auto-fills the **Environment name**).
4. Under **Platform**:

   * Select **Docker**
   * Change the platform branch to:
     `Docker running on 64bit Amazon Linux 2`
5. Under **Presets**, choose **Free tier eligible**.
6. Click **Next** to proceed to Service Access.
7. For service role:

   * Select **Create and use new service role**
   * Name it: `aws-elasticbeanstalk-service-role`
   * Choose EC2 instance profile: `aws-elasticbeanstalk-ec2-role` (auto-filled)
8. Click **Skip to Review** → **Submit**.
9. After deployment, click the **Domain link** to verify your live app.

---

## 📦 Update S3 Bucket Object Ownership

1. Go to **S3** in the AWS Console.
2. Select the **elasticbeanstalk** bucket for your environment.
3. Open the **Permissions** tab.
4. Click **Edit** next to **Object Ownership**.
5. Set:

   * **ACLs enabled**
   * **Bucket owner preferred** → **Object writer**
6. Acknowledge the warning and click **Save changes**.

---

## 🔐 Create IAM User for GitHub Secrets

1. In AWS, go to **IAM** → **Users** → **Add User**.
2. Enter a name, e.g., `docker-react-gh-ci`.
3. Select **Attach policies directly**.
4. Search for and enable:

   * `AdministratorAccess-AWSElasticBeanstalk`
5. Create the user.
6. Go to the user → **Security Credentials** → **Access Keys** → **Create access key**.
7. Choose **Command Line Interface (CLI)**.
8. Confirm and download the **Access Key ID** and **Secret Access Key**.

---

## 🔐 Add AWS Keys to GitHub Secrets

1. Navigate to your GitHub repo → **Settings** → **Secrets and variables** → **Actions** (or simply [navigate here](https://github.com/rob-bl8ke/docker-cli/settings/secrets/actions)).
2. Add the following secrets:

   * `AWS_ACCESS_KEY`: *Paste the Access Key ID*
   * `AWS_SECRET_KEY`: *Paste the Secret Access Key*
   * `DOCKER_USERNAME`: *Your Docker Hub username*
   * `DOCKER_PASSWORD`: *Your Docker Hub password or token*

💥 You do not need to create an environment or environment secrets. Doing so will cause a failure of the action without making additional changes to the workflow file.

---

## ⚙️ GitHub Actions Workflow for Deployment

Create the following file: `.github/workflows/deploy.yml`

```yaml
name: Deploy Frontend

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
      
      - run: docker build -t bobbyache/react-poc -f Dockerfile.dev .
      
      - run: docker run -e CI=true bobbyache/react-poc npm test

      - name: Generate deployment package
        run: zip -r deploy.zip . -x '*.git*'

      - name: Deploy to EB
        uses: einaregilsson/beanstalk-deploy@v18
        with:
          aws_access_key: ${{ secrets.AWS_ACCESS_KEY }}
          aws_secret_key: ${{ secrets.AWS_SECRET_KEY }}
          application_name: React-poc
          environment_name: React-poc-env
          existing_bucket_name: elasticbeanstalk-af-south-1-682033493001
          region: af-south-1
          version_label: ${{ github.sha }}
          deployment_package: deploy.zip
```

### 🔍 What It Does:

* **Trigger**: Executes when changes are pushed to `main`.
* **Build Steps**:

  1. Checks out your source code.
  2. Logs into Docker Hub.
  3. Builds the frontend Docker image.
  4. Runs tests inside the container.
  5. Zips the project into `deploy.zip`, excluding `.git`.
  6. Deploys the zip file to AWS Elastic Beanstalk using your secrets and S3 bucket.

---

## 🚀 Deploy the Application

1. Make a small update to `src/App.js` (e.g., greeting text).

2. Commit and push:

   ```bash
   git add .
   git commit -m "Trigger deployment"
   git push origin main
   ```

3. Watch your workflow on GitHub → **Actions tab**.

4. When complete:

   * You'll see a ✅ green checkmark.
   * Your AWS EB environment will show "Updating…"
   * When finished, health will be **green**, and your app will be live.

---

Let me know if you’d like a downloadable `.md` version or a version tailored for a specific audience (e.g. junior devs or students)!
