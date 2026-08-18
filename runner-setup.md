# GitHub Actions self-hosted runner setup

This repository includes a bundled runner in the `new-cicd-pipeline/actions-runner` folder. Use it to register a self-hosted runner that will execute the Terraform CI/CD workflow.

## 1. Register the runner

From a PowerShell terminal on your Windows runner host:

```powershell
cd "<repo-root>\new-cicd-pipeline\actions-runner"
./config.cmd --url https://github.com/<your-org>/<your-repo> --token <github-runner-token> --name di-b18-terraform-runner --labels terraform --work _work
```

If you want the runner to keep running as a service, start it with:

```powershell
./run.cmd
```

## 2. Required GitHub repository secrets

Add the following secrets in the repository settings under Settings > Secrets and variables > Actions:

- `ARM_CLIENT_ID`
- `ARM_CLIENT_SECRET`
- `ARM_TENANT_ID`
- `ARM_SUBSCRIPTION_ID`
- `AZURE_CREDENTIALS`

`AZURE_CREDENTIALS` should be the JSON object returned by an Azure service principal creation step, for example:

```bash
az ad sp create-for-rbac --name "di-b18-terraform" --role Contributor --scopes /subscriptions/<subscription-id>/resourceGroups/<resource-group>
```

## 3. Workflow behavior

The workflow in `.github/workflows/terraform-cicd.yml` does the following:

- runs on pull requests to `main`
- runs on pushes to `main`
- can also be triggered manually for a selected environment
- checks formatting, initializes Terraform, validates the configuration, and plans changes
- applies changes only when the manual dispatch input `apply` is enabled

## 4. Recommended check before production

Use a manual run with `apply=false` on the `prod` environment first. Once the plan looks correct, run the workflow again with `apply=true` for the production deployment.
