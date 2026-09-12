# Dynamic File Review System

An n8n workflow that automates pharmaceutical dossier compliance review (GPMI). It ingests a primary document plus reference documents, extracts and normalizes their content, runs an AI-driven audit against document-type-specific rules, and delivers a formatted PDF compliance report to the team.

## How it works

1. **Intake** — A webhook receives a job (primary document + reference documents) from the upstream system, along with metadata (product code, record IDs, pipeline key).
2. **Download & merge** — Files are downloaded and merged into a single item with one binary key per document.
3. **Extraction** — DOCX files are parsed locally (text, tables, images, embedded math); PDFs are sent to a companion Render.com extraction API. Images pulled from documents are analyzed with a vision model to describe charts, chromatograms, and tables.
4. **Dynamic type resolution** — The primary document's role (e.g. `SMPC`, `STABILITY_FP`, `PHYS_CHEM`) is detected from its filename and used to look up that document type's own review logic — splitting, system prompt, parsing, and formatting code — from a data table. Adding a new document type means adding a row to that table, not new workflow nodes.
5. **AI review** — A single generic agent runs the audit using the type-specific system prompt and document text, in one or more passes.
6. **Report generation** — The AI's structured output is parsed, formatted into a styled HTML report, and converted to PDF via the Render.com service.
7. **Delivery** — The PDF is uploaded to Zoho WorkDrive, the team is notified in Zoho Cliq, and the source record is updated with a link to the report.

## Requirements

- An n8n instance (self-hosted or cloud) with the following available:
  - `@n8n/n8n-nodes-langchain` (AI Agent, OpenAI Chat Model, OpenAI node)
  - n8n Data Tables (for document-type configuration and token storage)
- A companion PDF-extraction / HTML-to-PDF microservice (see `Wake Up Render` nodes — built for Render.com's free tier, which cold-starts after idling)
- Credentials configured in n8n for:
  - OpenAI (or compatible vision/chat model provider)
  - Zoho (WorkDrive, Cliq, Creator) OAuth tokens, stored in a Data Table

## Setup

1. Import `GPMI-Review-Main.json` into n8n.
2. Reconnect the credential references (OpenAI, Zoho) to your own n8n credentials — the exported file only contains internal credential IDs, not secrets.
3. Create the `GPMI Review Type Config` Data Table with one row per document type, each carrying that type's `splitCode`, `systemMessage`, `parserMergeCode`, `formatReportCode`, and `formatHtmlCode`.
4. Create the `Tokens` Data Table for Zoho OAuth tokens (Cliq, Creator, WorkDrive).
5. Point the webhook and the Render.com service URLs at your own infrastructure.

## Notes

- Test/example data (`pinData`) has been stripped from the exported workflow before publishing.
- Endpoint URLs and workspace identifiers in this export point to the original deployment — update them before running this yourself.

## License

Add a license of your choice (e.g. MIT) if you intend for others to reuse this.
