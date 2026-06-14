# Azure / Foundry quota log — what actually happened

Date: 2026-06-13. Goal: add a Foundry IQ knowledge base to the AI Support agent, grounded on 3 small local files (about 13 KB total). Region: Sweden Central. Subscription: "Azure subscription 1" (started as a 30-day free trial with about €172 credit, 25 days left).

This is a factual record of every step and every error message, in order. Times are from the screenshots.

## The files I tried to upload
- `evidence_base.md` — 4.67 KB
- `neurotype_reference.md` — 4.7 KB
- `support_techniques_reference.md` — 3.45 KB
- Total: about 13 KB of plain text. Three markdown files.

## Timeline

1. **Create knowledge source dialog.** All 3 files uploaded fine. Named it `support-knowledge`, wrote a description. The only required step left was to pick an embedding model.

2. **Picked `text-embedding-3-small`, clicked Deploy.** Result: `Model deployment error 715-123420: An error occurred. Please reach out to support for additional assistance.` Generic error, no detail.

3. **Tried `text-embedding-ada-002` instead.** Result: a clearer message: `'text-embedding-ada-002' isn't available due to insufficient quota.`

4. **Opened the deployment options for ada-002, changed deployment type to Global Standard.** Result: `Insufficient quota. The selected deployment type Global Standard with version 2 cannot be deployed to your current project due to insufficient quota.`

5. **Checked the Quota page (Token per minute, Sweden Central).** The only quota in use was two grok deployments at 50K/50K TPM each. There was **no embedding quota at all** in the region. Microsoft's own docs say generation-3 embedding models get a 350K TPM default per region. I had zero.

6. **Tried a non-OpenAI serverless embedding model (Cohere Embed v3 Multilingual)** to sidestep the OpenAI quota. Result: `Marketplace Subscription purchase eligibility check failed. The plan 'cohere-embed-v3-multilingual' can't be purchased using a free subscription. Please select a different subscription and try again.`

   So: the free 30-day trial is blocked from buying any marketplace model, even one I would pay for.

7. **Upgraded the free trial to pay-as-you-go** to get past the marketplace block. The upgrade kept my remaining €172 credit for the rest of the 30 days. Confirmed: `Successfully upgraded Subscription`. Picked the free Basic support plan.

8. **Retried Cohere multilingual, clicked Create.** Result: `Failed to create knowledge source. EmbeddingModel 'Cohere-embed-v3-multilingual' is not supported. Allowed models are: text-embedding-ada-002, text-embedding-3-small, text-embedding-3-large.`

   So Foundry IQ only accepts those three OpenAI embedding models. The Cohere route was never going to work, quota or not.

9. **Went back to `text-embedding-3-small` / ada-002 now that I was on pay-as-you-go.** Result: `Model deployment error 715-123420` again. Three times. The upgrade removed the marketplace block but did **not** grant any OpenAI embedding quota.

10. **Final, clean test (to be sure it wasn't a config mistake).** Downloaded Microsoft's official model-availability table. It confirms all three embedding models are available in Sweden Central under Global Standard (ada-002, 3-large, 3-small), and that under regional Standard, 3-small is NOT offered in Sweden Central (only ada-002 and 3-large). So I deployed `text-embedding-ada-002`, version 2, with deployment type set to **Global Standard** (the recommended, highest-rate-limit type), on the pay-as-you-go subscription. Result: `Insufficient quota. The selected deployment type Global Standard with version 2 cannot be deployed to your current project due to insufficient quota.`

   This rules out every config explanation. Right model, listed available in-region, correct deployment type, paid subscription. Quota was still zero.

## Where it ended
- Zero embedding quota in Sweden Central, before and after upgrading to pay-as-you-go.
- Foundry IQ refuses every embedding model except three OpenAI ones, all of which need quota I do not have.
- A managed vector index was required to "use" three small text files.

## The decision
Stopped fighting it. Grounded the agent directly with the three files in its instructions. No embeddings, no vector index, no quota. Documented the Foundry IQ knowledge base as architected and configured but blocked by a platform quota limit.

## Resolution: the quota exists in East US 2 (checked 2026-06-13 ~21:30)

The zero-quota problem was Sweden Central only. With "Show all" on the quota page, embedding quota in other regions:
- text-embedding-3-small, East US 2, Global Standard: 0/1,000,000 TPM (one million)
- text-embedding-3-small, East US 2, Standard: 0/350K TPM
- text-embedding-3-small, East US, Standard: 0/350K TPM
- (Sweden Central, where everything was built, had zero embedding quota of any type.)

So Foundry IQ IS buildable. Plan: build the knowledge base in East US 2 (the Foundry IQ cookbook's default agentic-retrieval region), deploy text-embedding-3-small or -large there, create the knowledge source from the 3 curated docs, create the knowledge base, and connect it to the knowledge-agent. This satisfies the hackathon's mandatory "integrate a Microsoft IQ layer" requirement.

## Facts worth keeping
- Generation-3 embedding models are documented to receive a 350K TPM default quota per region. Source: Microsoft Learn, "Azure OpenAI Quotas and Limits."
- A free trial subscription cannot purchase any Azure Marketplace model offer.
- Upgrading to pay-as-you-go does not add OpenAI quota. Quota is a separate per-region, per-model, per-deployment-type allocation.
