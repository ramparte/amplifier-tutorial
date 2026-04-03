---
id: provider-azure
type: provider
title: "Azure OpenAI Provider"
---

# Azure OpenAI Provider

Some teams can't send code to a third-party API endpoint — no matter how good the model is. Regulated industries, government contracts, internal compliance policies — the reasons vary but the constraint is the same: data must stay within your organization's control boundary. Azure OpenAI gives you the same GPT models you'd get from OpenAI directly, but running inside Microsoft's enterprise cloud with all the governance tooling Azure provides. If your company already lives in Azure, this is often the path of least resistance.

## Overview

The Azure OpenAI provider connects Amplifier to GPT models deployed through your Azure subscription. Instead of calling OpenAI's API directly, requests route through your own Azure resource endpoint — meaning traffic stays within your Azure tenant, subject to your network policies, your data residency rules, and your compliance certifications. The models themselves are identical. The difference is who controls the infrastructure they run on.

This matters when you need:

- **SOC 2, HIPAA, or FedRAMP compliance** — Azure's compliance portfolio covers these and more
- **Data residency** — deploy models in specific Azure regions to satisfy data sovereignty laws
- **VNet integration** — lock down the endpoint behind your virtual network, no public internet exposure
- **Azure AD authentication** — use managed identities instead of raw API keys
- **Enterprise billing** — route costs through your existing Azure agreement

## Setup

Azure OpenAI works differently from direct OpenAI. You don't just get an API key — you create an Azure OpenAI resource, deploy specific models to it, and then connect Amplifier to your resource's endpoint.

### Step 1: Create an Azure OpenAI Resource

In the Azure Portal, create an Azure OpenAI resource. You'll need to request access if you haven't already — Microsoft gates this behind an approval form.

Once provisioned, note your **endpoint** and **API key** from the resource's "Keys and Endpoint" page.

### Step 2: Deploy a Model

In Azure OpenAI Studio, create a deployment. Give it a name (e.g., `gpt-4o-deployment`) and select the model (e.g., GPT-4o). The deployment name is what you'll reference in Amplifier — not the model name directly.

### Step 3: Set Environment Variables

> Configure your Azure OpenAI credentials

```
[Tool: bash] export AZURE_OPENAI_API_KEY="your-azure-key-here"
[Tool: bash] export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com"
```

Or add them to your keys file for persistence:

```
[Tool: bash] cat ~/.amplifier/keys.env
AZURE_OPENAI_API_KEY=your-azure-key-here
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com
```

> Verify the endpoint is reachable

```
[Tool: bash] curl -s "$AZURE_OPENAI_ENDPOINT/openai/models?api-version=2024-06-01" \
  -H "api-key: $AZURE_OPENAI_API_KEY" | head -c 200
{"data":[{"id":"gpt-4o","object":"model"...
```

If you see model data, your credentials are working.

## Configuration

Azure OpenAI configuration requires a few fields that the direct OpenAI provider doesn't need — because the model is identified by deployment, not by name.

### Minimal Configuration

```yaml
# amplifier.yaml
provider:
  name: azure-openai
  endpoint: ${AZURE_OPENAI_ENDPOINT}
  deployment_name: gpt-4o-deployment
  api_version: "2024-06-01"
```

### Full Configuration

```yaml
provider:
  name: azure-openai
  api_key: ${AZURE_OPENAI_API_KEY}
  endpoint: https://your-resource.openai.azure.com
  deployment_name: gpt-4o-deployment
  api_version: "2024-06-01"
  temperature: 0.2
  max_tokens: 4096
```

### Configuration Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| **name** | — | Always `azure-openai` |
| **endpoint** | from env | Your Azure OpenAI resource URL |
| **deployment_name** | — | Name of your deployed model in Azure |
| **api_version** | `2024-06-01` | Azure API version string |
| **api_key** | from env | Azure OpenAI API key |
| **temperature** | `1.0` | Randomness (0.0–2.0, lower = more focused) |
| **max_tokens** | `4096` | Maximum tokens in the response |

The `api_version` field is Azure-specific — Microsoft versions their API independently from OpenAI. Use the latest stable version unless you have a reason to pin an older one.

### Azure AD Authentication

For environments that don't use API keys, Azure OpenAI supports managed identity authentication:

```yaml
provider:
  name: azure-openai
  endpoint: ${AZURE_OPENAI_ENDPOINT}
  deployment_name: gpt-4o-deployment
  api_version: "2024-06-01"
  auth_method: azure_ad
```

This uses your Azure CLI login or managed identity token — no API key needed. Ideal for CI/CD pipelines and production deployments where key rotation is a concern.

## Models

Azure OpenAI offers the same models as direct OpenAI — but you deploy them individually. Each deployment hosts one model at one version.

### Common Deployments

| Deployment | Model | Use Case |
|------------|-------|----------|
| `gpt-4o-deployment` | GPT-4o | General coding, vision, multimodal |
| `gpt-4o-mini-deployment` | GPT-4o-mini | Fast tasks, bulk operations |
| `o3-deployment` | o3 | Complex reasoning, hard debugging |

You choose the model when creating the deployment in Azure. Amplifier references the deployment name, not the underlying model name.

### Multi-Deployment Routing

You can create multiple deployments and route between them:

```yaml
providers:
  azure-openai:
    api_key: ${AZURE_OPENAI_API_KEY}
    endpoint: ${AZURE_OPENAI_ENDPOINT}
    api_version: "2024-06-01"

routing:
  coding: azure-openai/gpt-4o-deployment
  fast: azure-openai/gpt-4o-mini-deployment
  reasoning: azure-openai/o3-deployment
  vision: azure-openai/gpt-4o-deployment
```

This keeps everything within Azure while still leveraging Amplifier's routing matrix. Every request stays inside your compliance boundary.

## Azure vs Direct OpenAI

When should you use Azure OpenAI instead of the direct OpenAI API?

**Choose Azure OpenAI when:**
- Your organization mandates Azure for all cloud services
- You need compliance certifications (SOC 2, HIPAA, FedRAMP)
- Data residency requirements dictate where inference happens
- You want VNet-level network isolation
- Enterprise billing through an existing Azure agreement matters
- Azure AD / managed identity is your authentication standard

**Choose direct OpenAI when:**
- You want the simplest possible setup (one API key, done)
- You need access to the newest models on day one (Azure lags slightly)
- You don't have enterprise compliance requirements
- You want to avoid Azure resource management overhead

The models are the same. The code they produce is the same. The difference is entirely about infrastructure control, compliance posture, and how your organization manages cloud resources.

## Tips

- **Pin your API version.** Azure versions their API independently. Pinning avoids surprise breaking changes when Microsoft rolls out updates.
- **One deployment per model.** Each Azure deployment serves one model. Create separate deployments for GPT-4o, GPT-4o-mini, and any reasoning models you need.
- **Watch regional availability.** Not all models are available in all Azure regions. Check the Azure OpenAI model availability matrix before choosing a region.
- **Use managed identity in production.** API keys work for development, but production environments should use Azure AD with managed identities — no secrets to rotate.
- **Mind the deployment quota.** Azure applies per-deployment token-per-minute limits. If you're hitting 429 errors, increase quota in the Azure Portal or spread load across deployments.
- **Model updates lag slightly.** When OpenAI releases a new model, Azure availability follows days to weeks later. If bleeding-edge matters, use direct OpenAI for experimentation and Azure for production.

## Next Steps

- See the [Provider Index](./index.md) for multi-provider routing across Azure and other providers
- Compare with the [OpenAI Provider](./openai.md) to understand the direct API alternative
- Explore [Community Providers](./community.md) for AWS Bedrock if your enterprise runs on AWS instead
