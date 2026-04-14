# 🏗️ TP: Terraform

> \*\*Level:\*\* Engineering Students
> \*\*Duration:\*\* 3 hours
> \*\*OS:\*\* Windows (with Linux installation steps included)
> \*\*Prerequisites:\*\* Completed the Kubernetes TP (you know Pods, Deployments, Services, YAML manifests)
> \*\*AWS:\*\* You have an AWS Academy account via Vocareum

\---

## 📋 Table of Contents

1. [Introduction — What is Terraform?](#1--introduction--what-is-terraform)
2. [Step 1 — Install Terraform](#step-1--install-terraform)
3. [Step 2 — Terraform Basics: Your First Project](#step-2--terraform-basics-your-first-project)
4. [Step 3 — Manage Kubernetes Locally with Terraform](#step-3--manage-kubernetes-locally-with-terraform)
5. [Step 4 — Variables \& Outputs: Clean Code](#step-4--variables--outputs-clean-code)
6. [Step 5 — Deploy a Full App on Local K8s with Terraform](#step-5--deploy-a-full-app-on-local-k8s-with-terraform)
7. [Step 6 — Introduction to AWS with Terraform](#step-6--introduction-to-aws-with-terraform)
8. [Step 7 — Deploy a Simple App on AWS](#step-7--deploy-a-simple-app-on-aws)
9. [Step 8 — S3: Store Files in the Cloud](#step-8--s3-store-files-in-the-cloud)
10. [Step 9 — Full AWS App: EC2 + Security Group + S3](#step-9--full-aws-app-ec2--security-group--s3)
11. [Step 10 — Terraform State \& Best Practices](#step-10--terraform-state--best-practices)
12. [Cheat Sheet](#-cheat-sheet)

\---

## 1 — Introduction — What is Terraform?

### The Problem

In the Kubernetes TP, you created YAML files and ran `kubectl apply`. That works great for **one cluster**. But what happens when:

* You need to create the **cluster itself** (machines, networks, load balancers)?
* You need to create **cloud resources** (AWS EC2 servers, databases, S3 buckets)?
* You want to **reproduce the exact same infrastructure** on another environment?
* Someone accidentally deletes a server — how do you recreate everything?

**kubectl manages what runs *inside* a cluster. Terraform manages the infrastructure <i>underneath</i> and <i>around</i> it.**

### The Solution: Terraform

Terraform is an **Infrastructure as Code (IaC)** tool. You describe your infrastructure in files, and Terraform creates/updates/destroys it for you.

```
Without Terraform:                  With Terraform:

  Click around in AWS Console         Write a .tf file:
  Create a server... click click      "I want 2 servers, 1 database"
  Create a database... click click
  Configure networking... click       Run: terraform apply
                                      ✅ Everything created automatically
  Someone deletes something?
  Good luck recreating it 😱          Run: terraform apply again
                                      ✅ Recreated identically
```

### Key Concepts (5 words to remember)

|Concept|What is it?|Analogy|
|-|-|-|
|**Provider**|A plugin that talks to a platform (AWS, K8s, Azure)|A translator 🌐|
|**Resource**|Something you want to create (server, bucket, Pod)|A building block 🧱|
|**State**|Terraform's memory of what it created|A blueprint of your city 📐|
|**Plan**|A preview of what Terraform will do|An architect's drawing ✏️|
|**Apply**|Actually create/update the infrastructure|Construction day 🏗️|

### How It All Fits Together

```
                    You write
                    .tf files
                       │
                       ▼
              ┌─── Terraform ───┐
              │                 │
              │  1. terraform   │
              │     plan        │──► "Here's what I'll do" (preview)
              │                 │
              │  2. terraform   │
              │     apply       │──► Actually creates resources
              │                 │
              │  3. terraform   │
              │     destroy     │──► Deletes everything cleanly
              │                 │
              └────────┬────────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
    ┌──────────┐ ┌──────────┐ ┌──────────┐
    │   AWS    │ │ Kubernetes│ │  Azure   │
    │ Provider │ │ Provider  │ │ Provider │
    └──────────┘ └──────────┘ └──────────┘
          │            │            │
          ▼            ▼            ▼
      EC2, S3,     Pods, Svcs,   VMs, Storage
      VPC...       Deploys...    ...
```

### kubectl vs Terraform

||kubectl|Terraform|
|-|-|-|
|What it manages|Resources **inside** a K8s cluster|Any infrastructure (cloud, K8s, DNS...)|
|Language|YAML|HCL (HashiCorp Configuration Language)|
|State tracking|Cluster stores the state|Local file (`terraform.tfstate`)|
|Providers|Kubernetes only|3000+ (AWS, Azure, GCP, K8s, Docker...)|
|Best for|Day-to-day K8s operations|Creating \& managing infrastructure|
|Destroy|`kubectl delete`|`terraform destroy` (removes everything)|

> 📝 \*\*Key insight:\*\* You can use Terraform to create a Kubernetes cluster on AWS, \*and\* to deploy Pods into it. It does both!

\---

## Step 1 — Install Terraform

### 1.1 Install on Windows

**Option A: Using winget (recommended)**

```powershell
winget install HashiCorp.Terraform
```

Restart your PowerShell after installation.

**Option B: Using Chocolatey**

```powershell
choco install terraform
```

**Option C: Manual Installation**

1. Go to https://developer.hashicorp.com/terraform/install
2. Download the **Windows AMD64** zip
3. Extract to `C:\\terraform`
4. Add `C:\\terraform` to your **PATH**:

   * Search **"Environment Variables"** in Start Menu
   * Edit the **Path** variable (User or System)
   * Add `C:\\terraform`
   * Click OK
5. Restart PowerShell

### 1.2 Install on Linux (Ubuntu/Debian)

```bash
# Add HashiCorp GPG key and repository
sudo apt-get update \&\& sudo apt-get install -y gnupg software-properties-common curl
wget -O- https://apt.releases.hashicorp.com/gpg | \\
  gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

echo "deb \[signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \\
  https://apt.releases.hashicorp.com $(lsb\_release -cs) main" | \\
  sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt install terraform
```

**On Fedora/RHEL:**

```bash
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
sudo dnf install terraform
```

**Manual (any Linux distro):**

```bash
TERRAFORM\_VERSION="1.14.8"
curl -LO "https://releases.hashicorp.com/terraform/${TERRAFORM\_VERSION}/terraform\_${TERRAFORM\_VERSION}\_linux\_amd64.zip"
unzip "terraform\_${TERRAFORM\_VERSION}\_linux\_amd64.zip"
sudo mv terraform /usr/local/bin/
rm "terraform\_${TERRAFORM\_VERSION}\_linux\_amd64.zip"
```

### 1.3 Verify Installation

```powershell
terraform -version
```

Expected:

```
Terraform v1.x.x
```

> ✅ \*\*Terraform is installed!\*\*

\---

## Step 2 — Terraform Basics: Your First Project

Let's learn the core workflow with a simple example that doesn't need any cloud account.

### 2.1 The Terraform Workflow

Every Terraform project follows the same 3 steps:

```
terraform init     →  Download providers (plugins)
terraform plan     →  Preview what will happen
terraform apply    →  Do it for real
terraform destroy  →  Clean everything up
```

### 2.2 Create Your First Project

Create a folder:

```powershell
mkdir C:\\terraform-tp
cd C:\\terraform-tp
mkdir step2-basics
cd step2-basics
```

**On Linux:**

```bash
mkdir -p \~/terraform-tp/step2-basics
cd \~/terraform-tp/step2-basics
```

Create `main.tf`:

```hcl
# This tells Terraform which providers we need
terraform {
  required\_providers {
    random = {
      source  = "hashicorp/random"
      version = ">= 3.0.0"
    }
  }
}

# Create a random pet name (like "happy-panda")
resource "random\_pet" "my\_pet" {
  length    = 2
  separator = "-"
}

# Create a random password
resource "random\_password" "my\_password" {
  length  = 16
  special = true
}

# Show the results
output "pet\_name" {
  value = random\_pet.my\_pet.id
}

output "password" {
  value     = random\_password.my\_password.result
  sensitive = true
}
```

> 📝 \*\*Let's break it down:\*\*
> - `terraform {}` — declares which providers (plugins) we need
> - `resource "type" "name"` — something we want to create
> - `output` — values to display after `apply`
> - `sensitive = true` — hides the value in logs (good for passwords)

### 2.3 Init — Download Providers

```powershell
terraform init
```

```
Initializing the backend...
Initializing provider plugins...
- Installing hashicorp/random v3.x.x...
- Installed hashicorp/random v3.x.x

Terraform has been successfully initialized!
```

> 📝 This created a `.terraform/` folder (like `node\_modules` in JavaScript).

### 2.4 Plan — Preview Changes

```powershell
terraform plan
```

```
Terraform will perform the following actions:

  # random\_password.my\_password will be created
  + resource "random\_password" "my\_password" {
      + length  = 16
      + special = true
      + result  = (sensitive value)
    }

  # random\_pet.my\_pet will be created
  + resource "random\_pet" "my\_pet" {
      + length    = 2
      + separator = "-"
      + id        = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.
```

> 📝 The `+` means "will be created". Nothing has happened yet — this is just a preview!

### 2.5 Apply — Create Resources

```powershell
terraform apply
```

Terraform asks for confirmation:

```
Do you want to perform these actions?
  Enter a value: yes
```

Type `yes` and press Enter.

```
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

password = <sensitive>
pet\_name = "happy-panda"
```

🎉 You've created your first Terraform resources!

To see the password:

```powershell
terraform output password
```

### 2.6 State — Terraform's Memory

```powershell
# See what Terraform is tracking
terraform state list
```

```
random\_password.my\_password
random\_pet.my\_pet
```

```powershell
# See details of a resource
terraform state show random\_pet.my\_pet
```

> 📝 Terraform saved a file called `terraform.tfstate`. This is how it remembers what it created. \*\*Never edit this file manually!\*\*

### 2.7 Destroy — Clean Up

```powershell
terraform destroy
```

```
Destroy complete! Resources: 2 destroyed.
```

Everything is gone. Clean and reversible.

### 2.8 File Structure

```
step2-basics/
├── main.tf              ← your code (you write this)
├── terraform.tfstate    ← state file (Terraform manages this)
├── .terraform/          ← downloaded providers (like node\_modules)
└── .terraform.lock.hcl  ← version lock (like package-lock.json)
```

> ⚠️ \*\*Important:\*\* Add `terraform.tfstate`, `.terraform/` to `.gitignore` in real projects!

> ✅ \*\*Checkpoint:\*\* You understand the Terraform workflow: init → plan → apply → destroy!

\---

## Step 3 — Manage Kubernetes Locally with Terraform

Now let's connect Terraform to your **local Kubernetes cluster** (Docker Desktop).

### 3.1 Why Use Terraform for Kubernetes?

You already know `kubectl apply -f deployment.yaml`. Why use Terraform too?

||kubectl + YAML|Terraform|
|-|-|-|
|Tracks state|❌ No (cluster does)|✅ Yes (state file)|
|Destroy all at once|❌ Manual|✅ `terraform destroy`|
|Mix K8s + cloud|❌ K8s only|✅ Create VMs + deploy Pods in one tool|
|Dependencies|❌ Manual ordering|✅ Automatic (Terraform figures it out)|

### 3.2 Setup

Create a new folder:

```powershell
mkdir C:\\terraform-tp\\step3-k8s
cd C:\\terraform-tp\\step3-k8s
```

**On Linux:**

```bash
mkdir -p \~/terraform-tp/step3-k8s
cd \~/terraform-tp/step3-k8s
```

### 3.3 Create Namespace + Deployment + Service

Create `main.tf`:

```hcl
# Tell Terraform to use the Kubernetes provider
terraform {
  required\_providers {
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "\~> 2.27"
    }
  }
}

# Configure the provider to talk to your local cluster
provider "kubernetes" {
  config\_path = "\~/.kube/config"
}

# ── Namespace ──
resource "kubernetes\_namespace" "tp" {
  metadata {
    name = "terraform-tp"
  }
}

# ── Deployment ──
resource "kubernetes\_deployment" "nginx" {
  metadata {
    name      = "nginx"
    namespace = kubernetes\_namespace.tp.metadata\[0].name
  }

  spec {
    replicas = 3

    selector {
      match\_labels = {
        app = "nginx"
      }
    }

    template {
      metadata {
        labels = {
          app = "nginx"
        }
      }

      spec {
        container {
          name  = "nginx"
          image = "nginx:latest"

          port {
            container\_port = 80
          }
        }
      }
    }
  }
}

# ── Service ──
resource "kubernetes\_service" "nginx" {
  metadata {
    name      = "nginx-service"
    namespace = kubernetes\_namespace.tp.metadata\[0].name
  }

  spec {
    selector = {
      app = "nginx"
    }

    port {
      port        = 80
      target\_port = 80
      node\_port   = 30090
    }

    type = "NodePort"
  }
}
```

> 📝 \*\*Notice the HCL syntax vs YAML:\*\*
>
> ```
> YAML (kubectl):              HCL (Terraform):
>
> kind: Deployment             resource "kubernetes\_deployment" "nginx" {
> metadata:                      metadata {
>   name: nginx                    name = "nginx"
> spec:                          }
>   replicas: 3                  spec {
>                                  replicas = 3
> ```
>
> Different syntax, same result!

### 3.4 Deploy

```powershell
terraform init
terraform plan
terraform apply
```

Type `yes` when prompted.

Verify with kubectl:

```powershell
kubectl get all -n terraform-tp
```

```
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-xxxxx-aaaaa        1/1     Running   0          10s
pod/nginx-xxxxx-bbbbb        1/1     Running   0          10s
pod/nginx-xxxxx-ccccc        1/1     Running   0          10s

NAME                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)
service/nginx-service   NodePort   10.96.xxx.xxx   <none>        80:30090/TCP

NAME                    READY   UP-TO-DATE   AVAILABLE
deployment.apps/nginx   3/3     3            3
```

Open http://localhost:30090 → **Nginx welcome page!** 🎉

### 3.5 Change Something

Edit `main.tf` — change `replicas = 3` to `replicas = 5`:

```hcl
    replicas = 5
```

```powershell
terraform plan
```

```
  # kubernetes\_deployment.nginx will be updated in-place
  \~ resource "kubernetes\_deployment" "nginx" {
      \~ spec {
          \~ replicas = 3 -> 5
        }
    }

Plan: 0 to add, 1 to change, 0 to destroy.
```

> 📝 Terraform shows you \*\*exactly\*\* what will change before doing it.

```powershell
terraform apply
```

Check:

```powershell
kubectl get pods -n terraform-tp
```

5 Pods running!

### 3.6 Destroy Everything

```powershell
terraform destroy
```

```
Destroy complete! Resources: 3 destroyed.
```

```powershell
kubectl get all -n terraform-tp
```

Everything is gone — namespace, deployment, service, pods. **One command to destroy it all.**

> ✅ \*\*Checkpoint:\*\* You can manage Kubernetes resources with Terraform!

\---

## Step 4 — Variables \& Outputs: Clean Code

Hardcoding values is bad. Let's use **variables**.

### 4.1 Setup

```powershell
mkdir C:\\terraform-tp\\step4-variables
cd C:\\terraform-tp\\step4-variables
```

**On Linux:**

```bash
mkdir -p \~/terraform-tp/step4-variables
cd \~/terraform-tp/step4-variables
```

### 4.2 Create the Files

Terraform projects typically have 3 files:

```
project/
├── main.tf          ← resources
├── variables.tf     ← input variables (what you can customize)
└── outputs.tf       ← output values (what Terraform tells you)
```

Create `variables.tf`:

```hcl
variable "namespace\_name" {
  description = "The Kubernetes namespace to create"
  type        = string
  default     = "my-app"
}

variable "app\_name" {
  description = "The application name"
  type        = string
  default     = "web"
}

variable "replicas" {
  description = "Number of Pod replicas"
  type        = number
  default     = 2
}

variable "container\_image" {
  description = "Docker image to deploy"
  type        = string
  default     = "nginx:latest"
}

variable "node\_port" {
  description = "NodePort to expose the service"
  type        = number
  default     = 30100
}
```

Create `main.tf`:

```hcl
terraform {
  required\_providers {
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "\~> 2.27"
    }
  }
}

provider "kubernetes" {
  config\_path = "\~/.kube/config"
}

resource "kubernetes\_namespace" "app" {
  metadata {
    name = var.namespace\_name
  }
}

resource "kubernetes\_deployment" "app" {
  metadata {
    name      = var.app\_name
    namespace = kubernetes\_namespace.app.metadata\[0].name
  }

  spec {
    replicas = var.replicas

    selector {
      match\_labels = {
        app = var.app\_name
      }
    }

    template {
      metadata {
        labels = {
          app = var.app\_name
        }
      }

      spec {
        container {
          name  = var.app\_name
          image = var.container\_image

          port {
            container\_port = 80
          }
        }
      }
    }
  }
}

resource "kubernetes\_service" "app" {
  metadata {
    name      = "${var.app\_name}-service"
    namespace = kubernetes\_namespace.app.metadata\[0].name
  }

  spec {
    selector = {
      app = var.app\_name
    }

    port {
      port        = 80
      target\_port = 80
      node\_port   = var.node\_port
    }

    type = "NodePort"
  }
}
```

> 📝 \*\*Notice:\*\* `var.replicas` replaces the hardcoded `3`. `"${var.app\_name}-service"` uses string interpolation (like template literals in JavaScript).

Create `outputs.tf`:

```hcl
output "namespace" {
  value = kubernetes\_namespace.app.metadata\[0].name
}

output "app\_url" {
  value = "http://localhost:${var.node\_port}"
}

output "pod\_count" {
  value = var.replicas
}
```

### 4.3 Deploy with Default Values

```powershell
terraform init
terraform apply
```

```
Outputs:

app\_url   = "http://localhost:30100"
namespace = "my-app"
pod\_count = 2
```

Open http://localhost:30100 → Nginx! 🎉

### 4.4 Deploy with Custom Values

You can override variables in several ways:

**Option 1: Command-line flags**

```powershell
terraform apply -var="replicas=5" -var="app\_name=frontend"
```

**Option 2: A `.tfvars` file (best for projects)**

Create `dev.tfvars`:

```hcl
namespace\_name  = "development"
app\_name        = "dev-web"
replicas        = 1
container\_image = "nginx:1.25"
node\_port       = 30200
```

```powershell
terraform apply -var-file="dev.tfvars"
```

> 📝 This is how you manage \*\*multiple environments\*\* (dev, staging, prod) with the same code!

### 4.5 Clean Up

```powershell
terraform destroy
```

> ✅ \*\*Checkpoint:\*\* You can write reusable Terraform code with variables and outputs!

\---

## Step 5 — Deploy a Full App on Local K8s with Terraform

Let's recreate the WordPress + MySQL stack from the Kubernetes TP — but entirely in Terraform.

### 5.1 Setup

```powershell
mkdir C:\\terraform-tp\\step5-wordpress
cd C:\\terraform-tp\\step5-wordpress
```

**On Linux:**

```bash
mkdir -p \~/terraform-tp/step5-wordpress
cd \~/terraform-tp/step5-wordpress
```

### 5.2 The Complete Configuration

Create `variables.tf`:

```hcl
variable "namespace" {
  default = "wordpress"
}

variable "mysql\_password" {
  description = "MySQL root password"
  type        = string
  sensitive   = true
  default     = "rootpass123"
}

variable "wp\_db\_password" {
  description = "WordPress DB user password"
  type        = string
  sensitive   = true
  default     = "wppass123"
}
```

Create `main.tf`:

```hcl
terraform {
  required\_providers {
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "\~> 2.27"
    }
  }
}

provider "kubernetes" {
  config\_path = "\~/.kube/config"
}

# ── Namespace ──
resource "kubernetes\_namespace" "wp" {
  metadata {
    name = var.namespace
  }
}

# ═══════════════════════════════════
# MYSQL
# ═══════════════════════════════════

resource "kubernetes\_secret" "mysql" {
  metadata {
    name      = "mysql-secret"
    namespace = kubernetes\_namespace.wp.metadata\[0].name
  }

  data = {
    MYSQL\_ROOT\_PASSWORD = var.mysql\_password
    MYSQL\_PASSWORD      = var.wp\_db\_password
  }
}

resource "kubernetes\_persistent\_volume\_claim" "mysql" {
  metadata {
    name      = "mysql-pvc"
    namespace = kubernetes\_namespace.wp.metadata\[0].name
  }

  spec {
    access\_modes = \["ReadWriteOnce"]

    resources {
      requests = {
        storage = "2Gi"
      }
    }
  }
}

resource "kubernetes\_deployment" "mysql" {
  metadata {
    name      = "mysql"
    namespace = kubernetes\_namespace.wp.metadata\[0].name
  }

  spec {
    replicas = 1

    selector {
      match\_labels = {
        app = "mysql"
      }
    }

    template {
      metadata {
        labels = {
          app = "mysql"
        }
      }

      spec {
        container {
          name  = "mysql"
          image = "mysql:8"

          port {
            container\_port = 3306
          }

          env {
            name = "MYSQL\_ROOT\_PASSWORD"
            value\_from {
              secret\_key\_ref {
                name = kubernetes\_secret.mysql.metadata\[0].name
                key  = "MYSQL\_ROOT\_PASSWORD"
              }
            }
          }

          env {
            name  = "MYSQL\_DATABASE"
            value = "wordpress"
          }

          env {
            name  = "MYSQL\_USER"
            value = "wpuser"
          }

          env {
            name = "MYSQL\_PASSWORD"
            value\_from {
              secret\_key\_ref {
                name = kubernetes\_secret.mysql.metadata\[0].name
                key  = "MYSQL\_PASSWORD"
              }
            }
          }

          volume\_mount {
            name       = "mysql-data"
            mount\_path = "/var/lib/mysql"
          }
        }

        volume {
          name = "mysql-data"

          persistent\_volume\_claim {
            claim\_name = kubernetes\_persistent\_volume\_claim.mysql.metadata\[0].name
          }
        }
      }
    }
  }
}

resource "kubernetes\_service" "mysql" {
  metadata {
    name      = "mysql-service"
    namespace = kubernetes\_namespace.wp.metadata\[0].name
  }

  spec {
    selector = {
      app = "mysql"
    }

    port {
      port        = 3306
      target\_port = 3306
    }

    type = "ClusterIP"
  }
}

# ═══════════════════════════════════
# WORDPRESS
# ═══════════════════════════════════

resource "kubernetes\_persistent\_volume\_claim" "wordpress" {
  metadata {
    name      = "wordpress-pvc"
    namespace = kubernetes\_namespace.wp.metadata\[0].name
  }

  spec {
    access\_modes = \["ReadWriteOnce"]

    resources {
      requests = {
        storage = "2Gi"
      }
    }
  }
}

resource "kubernetes\_deployment" "wordpress" {
  metadata {
    name      = "wordpress"
    namespace = kubernetes\_namespace.wp.metadata\[0].name
  }

  spec {
    replicas = 1

    selector {
      match\_labels = {
        app = "wordpress"
      }
    }

    template {
      metadata {
        labels = {
          app = "wordpress"
        }
      }

      spec {
        container {
          name  = "wordpress"
          image = "wordpress:latest"

          port {
            container\_port = 80
          }

          env {
            name  = "WORDPRESS\_DB\_HOST"
            value = "${kubernetes\_service.mysql.metadata\[0].name}:3306"
          }

          env {
            name  = "WORDPRESS\_DB\_USER"
            value = "wpuser"
          }

          env {
            name = "WORDPRESS\_DB\_PASSWORD"
            value\_from {
              secret\_key\_ref {
                name = kubernetes\_secret.mysql.metadata\[0].name
                key  = "MYSQL\_PASSWORD"
              }
            }
          }

          env {
            name  = "WORDPRESS\_DB\_NAME"
            value = "wordpress"
          }

          volume\_mount {
            name       = "wordpress-data"
            mount\_path = "/var/www/html"
          }
        }

        volume {
          name = "wordpress-data"

          persistent\_volume\_claim {
            claim\_name = kubernetes\_persistent\_volume\_claim.wordpress.metadata\[0].name
          }
        }
      }
    }
  }
}

resource "kubernetes\_service" "wordpress" {
  metadata {
    name      = "wordpress-service"
    namespace = kubernetes\_namespace.wp.metadata\[0].name
  }

  spec {
    selector = {
      app = "wordpress"
    }

    port {
      port        = 80
      target\_port = 80
      node\_port   = 30080
    }

    type = "NodePort"
  }
}
```

Create `outputs.tf`:

```hcl
output "wordpress\_url" {
  value = "http://localhost:30080"
}

output "namespace" {
  value = kubernetes\_namespace.wp.metadata\[0].name
}
```

### 5.3 Deploy

```powershell
terraform init
terraform apply
```

Watch Pods come up:

```powershell
kubectl get pods -n wordpress -w
```

Wait until both show `Running` and `1/1`.

Open http://localhost:30080 → **WordPress setup page!** 🎉

### 5.4 Compare: kubectl vs Terraform

```
kubectl approach (K8s TP):           Terraform approach:

  mysql.yaml          (54 lines)       main.tf    (one file, everything)
  wordpress.yaml      (42 lines)       variables.tf  (inputs)
  kubectl apply -f mysql.yaml          outputs.tf    (outputs)
  kubectl apply -f wordpress.yaml
                                       terraform apply  ← one command
  kubectl delete -f wordpress.yaml
  kubectl delete -f mysql.yaml         terraform destroy ← one command
```

Both work! But Terraform gives you **state tracking**, **dependency management**, and the ability to **mix cloud + K8s** in one project.

### 5.5 Clean Up

```powershell
terraform destroy
```

> ✅ \*\*Checkpoint:\*\* You can deploy full apps on local Kubernetes using Terraform!

\---

## Step 6 — Introduction to AWS with Terraform

Now let's move to the **cloud**! We'll use your **AWS Academy Vocareum** account.

### 6.1 How AWS Academy Vocareum Works

Your Vocareum Learner Lab gives you:

* **Temporary AWS credentials** (they expire when the lab session ends)
* A **limited set of AWS services** (enough for this TP!)
* **No billing** — it's covered by the academy
* A **pre-created IAM role** called `LabRole` (you can't create your own roles)

### 6.2 Get Your AWS Credentials

1. Go to your **AWS Academy** course on Vocareum
2. Click **"Start Lab"** (wait for the green circle 🟢)
3. Click **"AWS Details"** (or "Show" next to AWS)
4. You'll see:

   * `AWS\_ACCESS\_KEY\_ID`
   * `AWS\_SECRET\_ACCESS\_KEY`
   * `AWS\_SESSION\_TOKEN`

> ⚠️ \*\*Important:\*\* These credentials are \*\*temporary\*\*. If your lab session expires, you need to get new ones!

### 6.3 Set AWS Credentials

**On Windows (PowerShell):**

```powershell
$env:AWS\_ACCESS\_KEY\_ID="paste-your-access-key-here"
$env:AWS\_SECRET\_ACCESS\_KEY="paste-your-secret-key-here"
$env:AWS\_SESSION\_TOKEN="paste-your-session-token-here"
```

**On Windows (Command Prompt):**

```cmd
set AWS\_ACCESS\_KEY\_ID=paste-your-access-key-here
set AWS\_SECRET\_ACCESS\_KEY=paste-your-secret-key-here
set AWS\_SESSION\_TOKEN=paste-your-session-token-here
```

**On Linux/Mac:**

```bash
export AWS\_ACCESS\_KEY\_ID="paste-your-access-key-here"
export AWS\_SECRET\_ACCESS\_KEY="paste-your-secret-key-here"
export AWS\_SESSION\_TOKEN="paste-your-session-token-here"
```

> 📝 \*\*Tip:\*\* Create a script file so you can re-run it quickly when credentials expire.

**Windows — create `set-aws-creds.ps1`:**

```powershell
$env:AWS\_ACCESS\_KEY\_ID="your-key"
$env:AWS\_SECRET\_ACCESS\_KEY="your-secret"
$env:AWS\_SESSION\_TOKEN="your-token"
Write-Host "✅ AWS credentials set!"
```

Run it: `.\\set-aws-creds.ps1`

**Linux — create `set-aws-creds.sh`:**

```bash
#!/bin/bash
export AWS\_ACCESS\_KEY\_ID="your-key"
export AWS\_SECRET\_ACCESS\_KEY="your-secret"
export AWS\_SESSION\_TOKEN="your-token"
echo "✅ AWS credentials set!"
```

Run it: `source ./set-aws-creds.sh`

> ⚠️ \*\*Never commit credential scripts to Git!\*\* Add them to `.gitignore`.

### 6.4 Install AWS CLI (Optional but Useful)

**Windows:**

```powershell
winget install Amazon.AWSCLI
```

**Linux:**

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86\_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
rm -rf aws awscliv2.zip
```

### 6.5 Verify Credentials

```powershell
aws sts get-caller-identity
```

```json
{
    "UserId": "AROA...:user123",
    "Account": "123456789012",
    "Arn": "arn:aws:sts::123456789012:assumed-role/voclabs/..."
}
```

If you see this, your credentials work! ✅

### 6.6 Understanding Vocareum Limitations

|What you CAN do|What you CANNOT do|
|-|-|
|Create EC2 instances|Create IAM roles/policies|
|Create S3 buckets|Create VPCs (use the default one)|
|Create Security Groups|Use all regions (stick to `us-east-1`)|
|Use the pre-created `LabRole`|Create new IAM users|

> 📝 We'll use `us-east-1` and the pre-created `LabRole` for everything.

> ✅ \*\*Checkpoint:\*\* You have AWS credentials set up and working!

\---

## Step 7 — Deploy a Simple App on AWS

Let's create an **EC2 instance** (a virtual machine) running a web server.

### 7.1 Setup

```powershell
mkdir C:\\terraform-tp\\step7-aws-ec2
cd C:\\terraform-tp\\step7-aws-ec2
```

**On Linux:**

```bash
mkdir -p \~/terraform-tp/step7-aws-ec2
cd \~/terraform-tp/step7-aws-ec2
```

### 7.2 What We're Building

```
┌─── AWS Cloud (us-east-1) ──────────────────────┐
│                                                  │
│  ┌─── Security Group ────────────────────┐      │
│  │  Allow: port 22 (SSH), port 80 (HTTP) │      │
│  │                                        │      │
│  │  ┌─── EC2 Instance ──────────┐        │      │
│  │  │                            │        │      │
│  │  │  Amazon Linux 2023         │        │      │
│  │  │  t2.micro (free tier)      │        │      │
│  │  │  Running: Nginx web server │        │      │
│  │  │                            │        │      │
│  │  │  Public IP: x.x.x.x       │        │      │
│  │  └────────────────────────────┘        │      │
│  └────────────────────────────────────────┘      │
│                                                  │
└──────────────────────────────────────────────────┘
```

### 7.3 Write the Configuration

Create `variables.tf`:

```hcl
variable "region" {
  description = "AWS region"
  default     = "us-east-1"
}

variable "instance\_type" {
  description = "EC2 instance type"
  default     = "t2.micro"
}
```

Create `main.tf`:

```hcl
terraform {
  required\_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "\~> 4.67"
    }
  }
}

provider "aws" {
  region = var.region
}

# ── Look up the latest Amazon Linux 2023 AMI ──
data "aws\_ami" "amazon\_linux" {
  most\_recent = true
  owners      = \["amazon"]

  filter {
    name   = "name"
    values = \["al2023-ami-\*-x86\_64"]
  }

  filter {
    name   = "virtualization-type"
    values = \["hvm"]
  }
}

# ── Security Group (firewall rules) ──
resource "aws\_security\_group" "web\_sg" {
  name        = "web-server-sg"
  description = "Allow HTTP and SSH"

  # Allow SSH (port 22) from anywhere
  ingress {
    from\_port   = 22
    to\_port     = 22
    protocol    = "tcp"
    cidr\_blocks = \["0.0.0.0/0"]
  }

  # Allow HTTP (port 80) from anywhere
  ingress {
    from\_port   = 80
    to\_port     = 80
    protocol    = "tcp"
    cidr\_blocks = \["0.0.0.0/0"]
  }

  # Allow all outbound traffic
  egress {
    from\_port   = 0
    to\_port     = 0
    protocol    = "-1"
    cidr\_blocks = \["0.0.0.0/0"]
  }

  tags = {
    Name = "web-server-sg"
  }
}

# ── EC2 Instance ──
resource "aws\_instance" "web" {
  ami                    = data.aws\_ami.amazon\_linux.id
  instance\_type          = var.instance\_type
  vpc\_security\_group\_ids = \[aws\_security\_group.web\_sg.id]

  # This script runs when the instance first boots
  user\_data = <<-EOF
    #!/bin/bash
    dnf update -y
    dnf install -y nginx
    systemctl start nginx
    systemctl enable nginx
    echo "<h1>Hello from Terraform on AWS! 🚀</h1>" > /usr/share/nginx/html/index.html
    echo "<p>Instance: $(hostname)</p>" >> /usr/share/nginx/html/index.html
    echo "<p>Region: ${var.region}</p>" >> /usr/share/nginx/html/index.html
  EOF

  tags = {
    Name = "terraform-web-server"
  }
}
```

> 📝 \*\*What's new:\*\*
> - `data "aws\_ami"` — a \*\*data source\*\* that looks up information (doesn't create anything)
> - `aws\_security\_group` — a firewall (which ports are open)
> - `aws\_instance` — a virtual machine (EC2)
> - `user\_data` — a bash script that runs on first boot (installs Nginx)

Create `outputs.tf`:

```hcl
output "instance\_id" {
  value = aws\_instance.web.id
}

output "public\_ip" {
  value = aws\_instance.web.public\_ip
}

output "web\_url" {
  value = "http://${aws\_instance.web.public\_ip}"
}

output "ami\_used" {
  value = data.aws\_ami.amazon\_linux.id
}
```

### 7.4 Deploy

Make sure your AWS credentials are set (Step 6.3), then:

```powershell
terraform init
terraform plan
terraform apply
```

Type `yes`. Wait 1-2 minutes for the instance to boot.

```
Outputs:

instance\_id = "i-0abc123def456"
public\_ip   = "54.x.x.x"
web\_url     = "http://54.x.x.x"
ami\_used    = "ami-0abcdef1234567890"
```

Open the `web\_url` in your browser → **"Hello from Terraform on AWS! 🚀"** 🎉

> 📝 It may take 1-2 minutes after `apply` for Nginx to finish installing. Refresh the page if you see an error.

### 7.5 Explore in AWS Console

1. In Vocareum, click **"AWS Console"**
2. Go to **EC2 → Instances**
3. Find your `terraform-web-server` instance
4. See its public IP, security group, and status

### 7.6 Clean Up

```powershell
terraform destroy
```

> ⚠️ \*\*Always run `terraform destroy` when done\*\* to avoid using up your Vocareum credits!

> ✅ \*\*Checkpoint:\*\* You deployed a web server on AWS with Terraform!

\---

## Step 8 — S3: Store Files in the Cloud

**S3** (Simple Storage Service) is AWS's file storage. Think of it as a "Google Drive" for your applications.

### 8.1 Setup

```powershell
mkdir C:\\terraform-tp\\step8-s3
cd C:\\terraform-tp\\step8-s3
```

**On Linux:**

```bash
mkdir -p \~/terraform-tp/step8-s3
cd \~/terraform-tp/step8-s3
```

### 8.2 Create an S3 Bucket

Create `main.tf`:

> ⚠️ \*\*Vocareum Note:\*\* We use AWS provider version `\~> 4.67` because the v5 provider checks `s3:GetBucketObjectLockConfiguration`, which the student account does not have permission for.

```hcl
terraform {
  required\_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "\~> 4.67"
    }
    random = {
      source  = "hashicorp/random"
      version = ">= 3.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

# ── Random suffix to make the bucket name unique ──
resource "random\_pet" "bucket\_suffix" {
  length = 2
}

# ── S3 Bucket ──
resource "aws\_s3\_bucket" "website" {
  bucket        = "tp-terraform-${random\_pet.bucket\_suffix.id}"
  force\_destroy = true

  tags = {
    Name        = "TP Terraform Website"
    Environment = "dev"
  }
}

# ── Upload a file to the bucket ──
resource "aws\_s3\_object" "index" {
  bucket       = aws\_s3\_bucket.website.id
  key          = "index.html"
  content      = <<-EOF
    <!DOCTYPE html>
    <html>
    <head><title>Terraform S3 Demo</title></head>
    <body>
      <h1>🎉 This page is served from S3!</h1>
      <p>Bucket: ${aws\_s3\_bucket.website.bucket}</p>
      <p>Created with Terraform</p>
    </body>
    </html>
  EOF
  content\_type = "text/html"
}

# ── Upload another file ──
resource "aws\_s3\_object" "info" {
  bucket       = aws\_s3\_bucket.website.id
  key          = "data/info.json"
  content      = jsonencode({
    project = "terraform-tp"
    version = "1.0"
    author  = "student"
  })
  content\_type = "application/json"
}
```

Create `outputs.tf`:

```hcl
output "bucket\_name" {
  value = aws\_s3\_bucket.website.bucket
}

output "bucket\_arn" {
  value = aws\_s3\_bucket.website.arn
}
```

### 8.3 Deploy

```powershell
terraform init
terraform apply
```

```
Outputs:

bucket\_arn  = "arn:aws:s3:::tp-terraform-happy-panda"
bucket\_name = "tp-terraform-happy-panda"
```

### 8.4 Verify with AWS CLI

```powershell
# List your buckets
aws s3 ls

# List files in your bucket (replace with your bucket name from the output)
aws s3 ls s3://tp-terraform-happy-panda --recursive
```

```
2026-04-12 10:00:00        123 data/info.json
2026-04-12 10:00:00        200 index.html
```

```powershell
# Download and display a file
aws s3 cp s3://tp-terraform-happy-panda/index.html -
```

### 8.5 Clean Up

```powershell
terraform destroy
```

> 📝 `force\_destroy = true` on the bucket allows Terraform to delete the bucket even if it still contains files.

> ✅ \*\*Checkpoint:\*\* You can create and manage S3 storage with Terraform!

\---

## Step 9 — Full AWS App: EC2 + Security Group + S3

Let's combine everything into a more realistic project: a web server that reads files from S3.

### 9.1 Setup

```powershell
mkdir C:\\terraform-tp\\step9-full-app
cd C:\\terraform-tp\\step9-full-app
```

**On Linux:**

```bash
mkdir -p \~/terraform-tp/step9-full-app
cd \~/terraform-tp/step9-full-app
```

### 9.2 Architecture

```
┌─── AWS Cloud (us-east-1) ──────────────────────────────────┐
│                                                              │
│  ┌─── S3 Bucket ───────────┐                                │
│  │  index.html              │                                │
│  └──────────────────────────┘                                │
│              ▲                                                │
│              │  EC2 reads website files                       │
│              │                                                │
│  ┌─── LabRole (pre-created) ┐                                │
│  │  Allows EC2 to read S3   │                                │
│  └───────────┬──────────────┘                                │
│              │                                                │
│  ┌─── Security Group ─────────────────────────┐              │
│  │                                             │              │
│  │  ┌─── EC2 Instance ──────────────────┐     │              │
│  │  │  Amazon Linux 2023                 │     │              │
│  │  │  Nginx (serves website from S3)    │     │              │
│  │  │  Public IP → http://x.x.x.x       │     │              │
│  │  └────────────────────────────────────┘     │              │
│  └─────────────────────────────────────────────┘              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

> 📝 \*\*Vocareum Note:\*\* We use the pre-created `LabRole` and `LabInstanceProfile` instead of creating our own IAM role, because the student account does not have permission to create IAM resources.

### 9.3 The Configuration

Create `variables.tf`:

```hcl
variable "region" {
  default = "us-east-1"
}

variable "project\_name" {
  default = "tp-terraform-fullapp"
}
```

Create `main.tf`:

```hcl
terraform {
  required\_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "\~> 4.67"
    }
    random = {
      source  = "hashicorp/random"
      version = ">= 3.0"
    }
  }
}

provider "aws" {
  region = var.region
}

resource "random\_pet" "suffix" {
  length = 2
}

locals {
  bucket\_name = "${var.project\_name}-${random\_pet.suffix.id}"
}

# ═══════════════════════════════════
# S3 BUCKET + WEBSITE FILES
# ═══════════════════════════════════

resource "aws\_s3\_bucket" "website\_files" {
  bucket        = local.bucket\_name
  force\_destroy = true

  tags = {
    Project = var.project\_name
  }
}

resource "aws\_s3\_object" "index\_html" {
  bucket       = aws\_s3\_bucket.website\_files.id
  key          = "index.html"
  content      = <<-EOF
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <title>Terraform Full App</title>
      <style>
        body { font-family: Arial, sans-serif; max-width: 800px; margin: 50px auto;
               background: #1a1a2e; color: #eee; padding: 20px; }
        h1 { color: #e94560; }
        .card { background: #16213e; padding: 20px; border-radius: 10px;
                margin: 20px 0; border-left: 4px solid #e94560; }
        code { background: #0f3460; padding: 2px 8px; border-radius: 4px; }
      </style>
    </head>
    <body>
      <h1>🏗️ Terraform Full Stack App</h1>
      <div class="card">
        <h2>✅ This page was deployed with Terraform!</h2>
        <p>The HTML is stored in <code>S3</code> and served by <code>Nginx on EC2</code>.</p>
      </div>
      <div class="card">
        <h2>Architecture</h2>
        <p>📦 S3 Bucket → stores this HTML</p>
        <p>🖥️ EC2 Instance → runs Nginx, pulls files from S3</p>
        <p>🔒 Security Group → allows port 80 (HTTP)</p>
        <p>🔑 LabRole → lets EC2 access S3</p>
      </div>
      <div class="card">
        <h2>Info</h2>
        <p>Bucket: <code>${local.bucket\_name}</code></p>
        <p>Region: <code>${var.region}</code></p>
      </div>
    </body>
    </html>
  EOF
  content\_type = "text/html"
}

# ═══════════════════════════════════
# IAM — Use the pre-created LabRole
# ═══════════════════════════════════
# Vocareum provides LabRole and LabInstanceProfile.
# We CANNOT create our own IAM roles — the student
# account does not have iam:CreateRole permission.
# We just reference the existing profile.

data "aws\_iam\_instance\_profile" "lab\_profile" {
  name = "LabInstanceProfile"
}

# ═══════════════════════════════════
# SECURITY GROUP
# ═══════════════════════════════════

resource "aws\_security\_group" "web" {
  name        = "${var.project\_name}-sg"
  description = "Allow HTTP traffic"

  ingress {
    from\_port   = 80
    to\_port     = 80
    protocol    = "tcp"
    cidr\_blocks = \["0.0.0.0/0"]
  }

  egress {
    from\_port   = 0
    to\_port     = 0
    protocol    = "-1"
    cidr\_blocks = \["0.0.0.0/0"]
  }
}

# ═══════════════════════════════════
# EC2 INSTANCE
# ═══════════════════════════════════

data "aws\_ami" "amazon\_linux" {
  most\_recent = true
  owners      = \["amazon"]

  filter {
    name   = "name"
    values = \["al2023-ami-\*-x86\_64"]
  }

  filter {
    name   = "virtualization-type"
    values = \["hvm"]
  }
}

resource "aws\_instance" "web" {
  ami                    = data.aws\_ami.amazon\_linux.id
  instance\_type          = "t2.micro"
  vpc\_security\_group\_ids = \[aws\_security\_group.web.id]
  iam\_instance\_profile   = data.aws\_iam\_instance\_profile.lab\_profile.name

  user\_data = <<-EOF
    #!/bin/bash
    dnf update -y
    dnf install -y nginx
    systemctl start nginx
    systemctl enable nginx

    # Pull website from S3
    aws s3 cp s3://${aws\_s3\_bucket.website\_files.bucket}/index.html /usr/share/nginx/html/index.html
  EOF

  tags = {
    Name = "${var.project\_name}-server"
  }
}
```

Create `outputs.tf`:

```hcl
output "web\_url" {
  value = "http://${aws\_instance.web.public\_ip}"
}

output "instance\_id" {
  value = aws\_instance.web.id
}

output "bucket\_name" {
  value = aws\_s3\_bucket.website\_files.bucket
}

output "public\_ip" {
  value = aws\_instance.web.public\_ip
}
```

### 9.4 Deploy

```powershell
terraform init
terraform plan
terraform apply
```

Type `yes`. Wait 2-3 minutes for the instance to boot and pull files from S3.

```
Outputs:

bucket\_name = "tp-terraform-fullapp-happy-panda"
instance\_id = "i-0abc123def456"
public\_ip   = "54.x.x.x"
web\_url     = "http://54.x.x.x"
```

Open the `web\_url` in your browser → **Full styled page served from S3!** 🎉

> 📝 It may take 2-3 minutes after `apply` for Nginx to finish installing and pulling from S3. Refresh the page if you see an error.

### 9.5 What Just Happened?

Terraform created **5 resources** in the right order:

```
1. random\_pet.suffix              ← generate unique name
2. aws\_s3\_bucket.website\_files    ← create bucket
3. aws\_s3\_object.index\_html       ← upload HTML to bucket
4. aws\_security\_group.web         ← firewall rules
5. aws\_instance.web               ← virtual machine (uses all of the above)

+ 1 data source: aws\_iam\_instance\_profile.lab\_profile ← looked up LabRole
+ 1 data source: aws\_ami.amazon\_linux                 ← looked up AMI
```

Terraform figured out the **dependency order** automatically! 🧠

### 9.6 Update the Website

Change the HTML in the `aws\_s3\_object.index\_html` resource, then:

```powershell
terraform apply
```

SSH into the instance to pull the new file, or simply destroy and recreate:

```powershell
terraform destroy
terraform apply
```

### 9.7 Clean Up

```powershell
terraform destroy
```

> ⚠️ \*\*Always destroy your resources when done with the lab session!\*\*

> ✅ \*\*Checkpoint:\*\* You deployed a full multi-service app on AWS!

\---

## Step 10 — Terraform State \& Best Practices

### 10.1 The State File

After `terraform apply`, Terraform creates `terraform.tfstate`. This file remembers everything:

```powershell
# List tracked resources
terraform state list

# Show details of a resource
terraform state show aws\_instance.web
```

> ⚠️ \*\*Rules for the state file:\*\*
> - \*\*Never edit it manually\*\*
> - \*\*Never commit it to Git\*\* (it may contain secrets)
> - In teams, store it remotely (S3 + DynamoDB) — but that's beyond this TP

### 10.2 .gitignore for Terraform Projects

Create `.gitignore`:

```
# Terraform
.terraform/
\*.tfstate
\*.tfstate.backup
\*.tfvars
!example.tfvars

# Credentials
set-aws-creds.\*
```

### 10.3 Project Structure Best Practices

```
my-project/
├── main.tf              ← providers + resources
├── variables.tf         ← input variables
├── outputs.tf           ← output values
├── terraform.tfvars     ← variable values (not committed)
├── .gitignore           ← ignore state + secrets
└── README.md            ← documentation
```

### 10.4 Naming Conventions

```hcl
# ✅ Good: descriptive names
resource "aws\_instance" "web\_server" { ... }
resource "aws\_s3\_bucket" "app\_assets" { ... }

# ❌ Bad: generic names
resource "aws\_instance" "instance1" { ... }
resource "aws\_s3\_bucket" "bucket" { ... }
```

### 10.5 Use `terraform fmt`

```powershell
# Auto-format your .tf files
terraform fmt

# Check formatting without changing files
terraform fmt -check
```

### 10.6 Use `terraform validate`

```powershell
# Check for syntax errors
terraform validate
```

### 10.7 Vocareum-Specific Tips

|Tip|Why|
|-|-|
|Always use provider `\~> 4.67`|v5+ checks S3 object lock (Vocareum denies this)|
|Use `LabInstanceProfile` for IAM|You can't create IAM roles|
|Stick to `us-east-1`|Your lab is configured for this region|
|Run `terraform destroy` before lab ends|Avoid resource limits on next session|
|Re-export credentials after restart|Vocareum tokens expire with the session|

> ✅ \*\*Checkpoint:\*\* You know best practices for real Terraform projects!

\---

## 📋 Cheat Sheet

```powershell
# ── Workflow ──
terraform init                          # Download providers (run once)
terraform plan                          # Preview changes
terraform apply                         # Create/update infrastructure
terraform apply -auto-approve           # Skip the "yes" prompt
terraform destroy                       # Delete everything
terraform destroy -auto-approve         # Skip the "yes" prompt

# ── Code Quality ──
terraform fmt                           # Auto-format .tf files
terraform validate                      # Check for errors

# ── State ──
terraform state list                    # List all tracked resources
terraform state show <resource>         # Details of a resource
terraform output                        # Show all outputs
terraform output <name>                 # Show one output

# ── Variables ──
terraform apply -var="key=value"        # Override a variable
terraform apply -var-file="prod.tfvars" # Use a variable file

# ── AWS Credentials (Vocareum) ──
# PowerShell:
$env:AWS\_ACCESS\_KEY\_ID="..."
$env:AWS\_SECRET\_ACCESS\_KEY="..."
$env:AWS\_SESSION\_TOKEN="..."

# Linux/Mac:
export AWS\_ACCESS\_KEY\_ID="..."
export AWS\_SECRET\_ACCESS\_KEY="..."
export AWS\_SESSION\_TOKEN="..."

# ── Common Troubleshooting ──
# "Provider not found"      → Run: terraform init
# "Access Denied on S3"     → Use provider version \~> 4.67
# "Cannot create IAM role"  → Use data "aws\_iam\_instance\_profile" "lab\_profile"
# "Credentials expired"     → Get new creds from Vocareum, re-export them
# "State lock"              → Delete .terraform.lock.hcl and re-init
```

\---

## kubectl YAML → Terraform HCL Translation

|kubectl YAML|Terraform HCL|
|-|-|
|`kind: Deployment`|`resource "kubernetes\_deployment"`|
|`kind: Service`|`resource "kubernetes\_service"`|
|`kind: ConfigMap`|`resource "kubernetes\_config\_map"`|
|`kind: Secret`|`resource "kubernetes\_secret"`|
|`kind: PersistentVolumeClaim`|`resource "kubernetes\_persistent\_volume\_claim"`|
|`kind: Namespace`|`resource "kubernetes\_namespace"`|
|`kubectl apply -f .`|`terraform apply`|
|`kubectl delete -f .`|`terraform destroy`|

\---

## Docker Compose → kubectl → Terraform (Full Journey)

|Concept|Docker Compose|kubectl (K8s)|Terraform|
|-|-|-|-|
|Config file|`docker-compose.yml`|`.yaml` manifests|`.tf` files|
|Language|YAML|YAML|HCL|
|Start|`docker compose up`|`kubectl apply -f .`|`terraform apply`|
|Stop|`docker compose down`|`kubectl delete -f .`|`terraform destroy`|
|Scale|❌ Manual|`kubectl scale`|Change `replicas` → `apply`|
|State tracking|❌ No|Cluster-side|`terraform.tfstate`|
|Cloud resources|❌ No|❌ No (K8s only)|✅ Yes (AWS, Azure, GCP...)|
|Best for|Local dev|K8s operations|Infrastructure provisioning|

\---

> 🎉 \*\*Congratulations!\*\* You've gone from zero Terraform knowledge to:
> - Understanding Infrastructure as Code
> - Managing local Kubernetes with Terraform
> - Using variables and outputs for clean, reusable code
> - Deploying WordPress + MySQL via Terraform on local K8s
> - Creating AWS EC2 instances and S3 buckets
> - Building a full multi-service app on AWS (with Vocareum's LabRole)
> - Following best practices for real projects
>
> \*\*Bon courage !\*\* 🏗️

