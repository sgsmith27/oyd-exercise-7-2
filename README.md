# Exercise 7.2 — Multi-Env Layout and GitHub Environment Promotion

Curso: Optimizaciones y Desempeño — Cloud Deployment Automation

## Objetivo

Evolucionar un pipeline de Terraform desde una única validación monolítica hacia un flujo de CI/CD con:

* Validaciones independientes en Pull Requests
* Generación y publicación automática del Terraform Plan
* Gestión de artefactos
* Promoción entre ambientes
* Aprobación manual para despliegues en staging
* Uso de GitHub Environments

---

## Arquitectura del Pipeline

```text
Pull Request
│
├── terraform-fmt
├── terraform-validate
└── terraform-plan
      │
      ├── tfplan-dev artifact
      └── PR Comment

Merge to Main
│
├── apply-dev
│
└── apply-staging
       │
       └── Manual Approval
```

---

## Estructura del Repositorio

```text
.github/
└── workflows/
    └── terraform-cd.yml

infra/
├── provider.tf
├── main.tf
├── variables.tf
└── envs/
    ├── dev/
    │   ├── dev.tfvars
    │   └── backend-dev.hcl
    └── staging/
        ├── staging.tfvars
        └── backend-staging.hcl

bootstrap/
├── provider.tf
├── variables.tf
├── main.tf
└── outputs.tf

evidence/
└── pr-url.txt
```

---

## Backend Remoto

Bucket S3 utilizado para almacenar el estado remoto de Terraform:

```text
sgsmith27-oyd-exercise-7-2-tfstate
```

Versioning:

```text
Enabled
```

---

## GitHub Environments

### dev

Sin reglas de protección.

### staging

Configurado con:

* Required Reviewers
* Aprobación manual antes del despliegue

---

## Jobs del Workflow

### terraform-fmt

Verifica formato Terraform.

```bash
terraform fmt -check
```

### terraform-validate

Inicializa Terraform sin backend y valida la configuración.

```bash
terraform init -backend=false
terraform validate
```

### terraform-plan

Genera el plan para el ambiente dev.

```bash
terraform init -backend-config=envs/dev/backend-dev.hcl
terraform plan -var-file=envs/dev/dev.tfvars -out=tfplan
terraform show -no-color tfplan > plan.txt
```

Además:

* Publica artifact `tfplan-dev`
* Publica comentario automático en el Pull Request

### apply-dev

Despliega automáticamente al ambiente dev después del merge.

### apply-staging

Despliega al ambiente staging únicamente después de aprobación manual.

---

## Evidencia

### Pull Request

La URL del Pull Request utilizado para validar:

* terraform-fmt
* terraform-validate
* terraform-plan
* comentario automático del plan

se encuentra en:

```text
evidence/pr-url.txt
```
#### Comentarios del plan
#### Sin cambios
![PR 1 ](/evidence/pr1-not_changes.PNG)
#### Apicando cambios
![PR 1 ](/evidence/pr2-changes.PNG)

---

## Validación Exitosa

Se verificó:

* Ejecución de los tres status checks independientes
* Publicación automática del Terraform Plan en el Pull Request
* Generación del artifact tfplan-dev
* Despliegue automático a dev
* Aprobación manual para staging
* Despliegue exitoso a staging

---

## Conceptos Aplicados

* GitHub Actions
* Terraform CI/CD
* GitHub Environments
* Environment Promotion
* Required Reviewers
* Pull Request Automation
* Terraform Artifacts
* Remote State
* Multi-Environment Infrastructure
* Infrastructure as Code (IaC)
* Continuous Delivery
* Deployment Gates
* Terraform Plan Reviews

---

## Autor
Sergio Geovany García Smith

Carnet 2500813
