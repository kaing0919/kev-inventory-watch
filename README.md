# KEV Inventory Watch for n8n

Match new CISA Known Exploited Vulnerability catalog entries against a small software inventory and deliver a daily Discord digest. No LLM, paid search service, NVD API key or external database is required. n8n hosting may have its own costs; this project does not include hosting.

The workflow defaults to **inactive, preview only**. It contains no credentials, tracking or affiliate links.

## Who this helps

Small IT teams that already use n8n and Discord and want an auditable watchlist instead of a broad vulnerability-news feed. Matching uses explicit, exact vendor and product aliases. It does **not** detect installed software, compare affected versions or certify exposure. Verify candidate findings against vendor advisories before remediation.

## Set up

1. Import `workflow.json` into a current n8n instance. Keep it unpublished/inactive initially.
2. Edit **Configure inventory**. Each row needs a label, `vendorAliases` and `productAliases`. Use exact spellings from the [official KEV mirror](https://github.com/cisagov/kev-data). Matching normalizes case, whitespace and Unicode, but does not infer a vendor/product synonym or split combined product strings. The supplied Windows/FortiOS rows are examples, not your actual inventory.
3. Keep `dryRun: true`. Run **Manual preview** and inspect **Inspect preview or baseline**. It shows up to eight historical matches for a fresh workflow and sends nothing. A preview with no matches is possible; recheck your aliases.
4. In your own Discord server/channel, create a channel webhook and store it in an **n8n Discord Webhook credential**. Select that credential in **Discord alert**. The webhook URL is a secret; never paste it into the exported JSON or a public repository.
5. Set `dryRun: false`. Confirm the workflow timezone and schedule, then publish/activate. The first successful scheduled execution saves a quiet baseline and sends no historical alerts. Later newly added matches are delivered daily. Review historical matches during preview so the quiet baseline does not hide work you need to do.
6. Configure your instance's error workflow or execution monitoring. Failed catalog retrieval and delivery must surface as failed executions. Do not enable **Continue On Fail** on Discord. Keep a single active copy and avoid overlapping runs.

The state uses [workflow static data](https://docs.n8n.io/code/cookbook/builtin/get-workflow-static-data/). n8n saves it for successful production executions. Editor tests and standalone CLI previews do not establish persistent scheduled state. Changing inventory establishes a fresh baseline. Losing/resetting workflow state establishes a new baseline, so review the preview before resuming after recovery.

## Delivery and failure behavior

- An empty, malformed or count-mismatched catalog fails without acknowledging items.
- First live execution records all catalog CVE IDs as the baseline. New matches then alert once per successful workflow execution.
- State advances after every message in the batch succeeds. If a later part fails after earlier parts were sent, a retry can resend those earlier parts. Delivery is **at least once**, not exactly once.
- Preview mode does not alter state or contact Discord. It only reads the public CISA GitHub mirror.
- Messages are split below Discord's 2,000-character limit. Catalog fields and inventory labels cannot trigger `@` mentions.
- Alias matching deliberately favors precision over recall. Review unmatched products and aliases periodically. An existing CVE's changed metadata does not trigger a new-CVE alert.
- CISA due dates are catalog metadata about federal remediation deadlines; the workflow does not determine your organization's legal duties.

## Verification

`validation.json` records the actual checks and limits. Logic checks used synthetic positive/negative controls and a downloaded current official catalog. The workflow runtime preview is reported separately when completed. No Discord credential was supplied, so no external Discord message, durable production schedule, affiliate conversion, commission or income is claimed.

## Sources

- [CISA KEV data mirror](https://github.com/cisagov/kev-data)
- [n8n Discord node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.discord/)
- [n8n Discord credentials](https://docs.n8n.io/integrations/builtin/credentials/discord/)
- [n8n workflow static data](https://docs.n8n.io/code/cookbook/builtin/get-workflow-static-data/)

## License

MIT for the original workflow and documentation. CISA data retains its upstream public-domain status. n8n and Discord are third-party products; no endorsement is implied.

