# Design Decisions — Azure Healthcare Application Assessment

**Time spent:** approximately 3 hours. **Scope cut:** full production deployment verification, detailed multi-region disaster recovery, deeper cost modelling, and additional policy-as-code checks.

## Private data services

Azure SQL and Blob Storage are designed with public network access disabled and private endpoints. Private DNS zones and VNet links support name resolution from the application network. Private endpoint existence alone does not prove connectivity; DNS, connection state, routing, and authentication must also be verified.

## Compute and ingress

The design uses an API Web App and a separate background-worker Web App. Azure Front Door provides the public HTTPS entry point for the API, with origin health checks and routing. The API origin should restrict direct access to the intended Front Door profile. The worker has no intended public caller and should deny inbound traffic by default.

## Identity and secrets

The design favours managed identity and scoped RBAC for Storage rather than account keys. SQL is intended to use Microsoft Entra authentication instead of distributing SQL passwords. Application identities still require correctly scoped database permissions, and any runtime secret handling must be tested.

## Terraform and environments

The infrastructure is separated into reusable networking, data, compute, and edge components. Development, QA, and production are intended to share the same structure with environment-specific variables and sizing. Terraform state must be stored securely and must never be committed to a public repository.

## CI/CD evidence

GitHub Actions validation and plan workflows provide automated checks and a reviewable preview of infrastructure changes. Successful validation or plan runs should not be described as successful deployment. The deployment attempt encountered Azure service quota/region restrictions and an existing Front Door origin-group conflict.

## Cost and next steps

Always-on components such as App Service Plans, private endpoints, and Front Door may incur costs even at low traffic. Before a production rollout, I would confirm current pricing, assess sharing the Front Door profile across non-production environments, verify WAF and Private Link requirements, add policy guardrails, and perform end-to-end connectivity, monitoring, and recovery tests.
