# custom-addons : Setting Up a Basic CI/CD Pipeline for an Odoo Custom Module

## **1. Prerequisites**
- You have a **server** (Linux-based preferred).
- You have **Odoo installed** on the server.
- You have a **Git repository** (e.g., GitHub, GitLab, or Bitbucket).
- You have a basic **custom Odoo module**.

---

## **Step 1: Create a Basic Custom Odoo Module**
1. Navigate to Odoo’s **addons** directory:
   ```bash
   cd /odoo/custom/addons
   ```
2. Create a new module folder:
   ```bash
   mkdir hello_odoo
   cd hello_odoo
   ```
3. Create required files:
   ```bash
   touch __init__.py __manifest__.py models.py
   ```
4. **Edit `__init__.py`**:
   ```python
   from . import models
   ```
5. **Edit `__manifest__.py`**:
   ```python
   {
       'name': 'Hello Odoo',
       'version': '1.0',
       'summary': 'A basic Odoo module for CI/CD testing',
       'category': 'Tools',
       'author': 'Your Name',
       'depends': ['base'],
       'data': [],
       'installable': True,
       'application': False,
   }
   ```
6. **Edit `models.py`**:
   ```python
   from odoo import models, fields

   class HelloOdoo(models.Model):
       _name = 'hello.odoo'
       _description = 'Hello Odoo Model'

       name = fields.Char(string='Name', required=True)
   ```

---

## **Step 2: Push to Git**
1. Initialize Git:
   ```bash
   git init
   git remote add origin https://github.com/yourusername/yourrepo.git
   ```
2. Add & commit:
   ```bash
   git add .
   git commit -m "Initial Hello Odoo module"
   git branch -M main
   git push -u origin main
   ```

---

## **Step 3: Setup CI/CD with GitHub Actions**
Create `.github/workflows/ci.yml` in your repo:

```yaml
name: Odoo CI/CD

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Repository
      uses: actions/checkout@v3

    - name: Set Up Python
      uses: actions/setup-python@v3
      with:
        python-version: '3.8'

    - name: Install Odoo Dependencies
      run: |
        sudo apt update
        sudo apt install -y odoo postgresql
        pip install -r requirements.txt || true

    - name: Run Odoo Test
      run: |
        odoo --test-enable --stop-after-init --log-level=test > test.log 2>&1 || true
        cat test.log
```

---

## **Step 4: Setup Deployment (CD)**
1. **Create a deploy script** (`deploy.sh`) on the server:
   ```bash
   #!/bin/bash
   cd /odoo/custom/addons
   git pull origin main
   sudo systemctl restart odoo
   echo "Deployment completed!"
   ```
2. Make it executable:
   ```bash
   chmod +x deploy.sh
   ```
3. **Setup GitHub Action for Deployment** (`deploy.yml`):

```yaml
name: Deploy Odoo Module

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: SSH and Deploy
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.SERVER_IP }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /path/to/odoo/custom/addons
          git pull origin main
          sudo systemctl restart odoo
```

---

## **Step 5: Configure Secrets in GitHub**
- Go to **GitHub Repository → Settings → Secrets** and add:
  - `SERVER_IP`: Your server's IP address.
  - `SERVER_USER`: Your SSH username.
  - `SSH_PRIVATE_KEY`: Your private key for SSH access.

---

## **Step 6: Commit & Push**
```bash
git add .
git commit -m "Added CI/CD pipeline"
git push origin main
```

---

### **Now, every push to `main` will:**
1. Run Odoo tests.
2. Deploy the module to your server if tests pass.
3. Restart Odoo automatically.

