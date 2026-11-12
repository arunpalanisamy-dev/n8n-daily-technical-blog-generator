# Daily Technical Article Generator

An n8n workflow that automatically generates one technical article each day, formats the result as HTML, emails it for human review, and records article metadata in an n8n Data Table.

The workflow is configured with a daily 7:00 AM schedule, an Anthropic chat model with a 16,000-token output limit, Gmail for delivery, and an `article_history` Data Table to track previously generated articles and help rotate topics.

> **Important:** This README describes the uploaded workflow JSON, `Daily Technical Article Generator.json`. It does not imply that credentials, the n8n instance, or the external services are included in this repository.

## Contents

- [How it works](#how-it-works)
- [Features](#features)
- [Topic categories](#topic-categories)
- [Requirements](#requirements)
- [Setup](#setup)
- [Configuration](#configuration)
- [Run and operate the workflow](#run-and-operate-the-workflow)
- [Article output](#article-output)
- [Data stored in article_history](#data-stored-in-article_history)
- [Security and privacy](#security-and-privacy)
- [Troubleshooting](#troubleshooting)
- [Limitations and notes](#limitations-and-notes)
- [Contributing](#contributing)
- [License](#license)

## How it works

The workflow follows this sequence:

1. **Every Day at 7 AM** — starts the workflow once per day at the configured hour. The effective timezone comes from the n8n workflow/instance timezone configuration.
2. **Get Past Articles** — retrieves all rows from the `article_history` Data Table.
3. **Pick Category** — selects one of ten categories using the number of history rows with a non-empty title, and records the current date.
4. **Write Article** — asks the configured Anthropic chat model to create a complete article while avoiding titles already recorded for the selected category.
5. **Extract Metadata** — extracts the title, topic, category, date, article body, metadata, and quality-review text from the model response.
6. **Convert to HTML** — converts the article and review information from Markdown to HTML.
7. **Email Article for Review** — sends the formatted result using the configured Gmail account.
8. **Save to History** — after the email node succeeds, inserts the category, title, topic, and date into `article_history`.

The workflow JSON contains eight nodes, including the connected Anthropic Chat Model node.

## Features

- Scheduled daily article generation.
- Rotation across ten technical subject categories.
- Reads prior article history before selecting a topic.
- Prompt instructions request original, practical articles of approximately 1,000–1,500 words.
- Requests technical explanations, architecture/workflow discussion, implementation examples, applications, common pitfalls, best practices, conclusions, and references.
- Requests exactly five relevant Medium tags in the article metadata.
- Includes a quality-review section and editorial notes in the generated response.
- Extracts article fields from the model response for downstream processing.
- Converts Markdown content to HTML for email delivery.
- Sends the article to a configured Gmail recipient for human review.
- Records article metadata in an n8n Data Table.

## Topic categories

The workflow rotates through these categories:

1. Python
2. SQL
3. AI Agents
4. Generative AI
5. Machine Learning
6. Retrieval-Augmented Generation (RAG)
7. MLOps
8. Data Engineering
9. Deep Learning
10. n8n Automation

The category expression uses the count of history rows that have a title, modulo ten. It is not a strict per-category scheduler; see [Limitations and notes](#limitations-and-notes).

## Requirements

The workflow JSON does not include the n8n runtime or service credentials. Before importing and running it, you need:

- An n8n instance with the Schedule Trigger, Data Table, Set, Markdown, Gmail, and LangChain/Anthropic nodes available.
- An Anthropic model credential configured in n8n and permitted to use the model selected by this workflow (`claude-sonnet-5-5`).
- A Gmail OAuth2 credential configured in n8n, with permission to send email.
- An n8n Data Table named `article_history`.
- A workflow/instance timezone configured for the intended 7:00 AM schedule.

The exported workflow identifies its AI model as `claude-sonnet-5-5` and sets `maxTokensToSample` to `16000`. It does **not** use Ollama in the uploaded JSON.

## Setup

### 1. Import the workflow

1. Open your n8n instance.
2. From the workflows area, choose the option to import a workflow from a file.
3. Select `Daily Technical Article Generator.json`.
4. Open the imported workflow and review its nodes and connections.

Importing the JSON does not recreate credentials or guarantee that the external services are configured.

### 2. Configure the Anthropic credential

1. Open the **Anthropic Chat Model** node.
2. Select or create an Anthropic API credential that your n8n instance can use.
3. Confirm that the configured account/provider supports the model identifier `claude-sonnet-5-5`.
4. Save the node and test the model connection using your n8n instance's available test/execution options.

The exported JSON contains a credential reference/name, not the credential secret. You must configure the credential in the target n8n instance.

### 3. Configure Gmail

1. Open the **Email Article for Review** node.
2. Select or create a Gmail OAuth2 credential.
3. Confirm that the connected Google account is authorized to send mail.
4. Review the **Send To** field and change it if necessary.
5. Send a test message before enabling the scheduled workflow.

The uploaded workflow has a recipient address configured in this node. Review it before importing or sharing the workflow JSON.

### 4. Create the `article_history` Data Table

Create an n8n Data Table named exactly:

```text
article_history
```

Add these string columns:

| Column | Purpose |
| --- | --- |
| `category` | Selected topic category |
| `title` | Generated article title |
| `topic` | Specific article topic |
| `published_date` | Date value recorded by the workflow |

The workflow reads all rows from this table and inserts values into these four columns after the email node succeeds. Despite the column name `published_date`, the workflow records the generation date; it does not verify that the article was actually published on Medium.

### 5. Set the timezone

The Schedule Trigger specifies hour `7`, but the exported workflow JSON does not explicitly specify a timezone. Configure the workflow timezone or the n8n instance timezone to the timezone you intend to use. For a 7:00 AM India schedule, use:

```text
Asia/Kolkata
```

Verify the next execution time in n8n before relying on the schedule.

### 6. Test before activation

1. Confirm that the Anthropic and Gmail credentials are selected.
2. Confirm that `article_history` exists and has the required columns.
3. Review the recipient and schedule timezone.
4. Run a manual test.
5. Inspect the generated output, email formatting, and inserted Data Table row.
6. Confirm the workflow is saved and active when you are ready for scheduled runs.

## Configuration

The current values are defined in the workflow nodes:

| Setting | Current workflow configuration |
| --- | --- |
| Workflow name | `Daily Technical Article Generator` |
| Schedule | Daily at hour `7` |
| Timezone | Not explicitly stored in the exported JSON |
| AI node | Anthropic Chat Model |
| Model identifier | `claude-sonnet-5-5` |
| Maximum output tokens | `16000` |
| Article length requested by prompt | 1,000–1,500 words |
| Email service | Gmail |
| Email recipient | Configured in the Gmail node; review before use |
| History store | n8n Data Table named `article_history` |

There is no `.env` file or environment-variable configuration defined by the uploaded workflow JSON. Credentials are selected in n8n, not supplied as environment variables by this workflow.

### Changing the schedule

Edit the **Every Day at 7 AM** Schedule Trigger node. The JSON specifies `triggerAtHour: 7`. The timezone must be configured separately in n8n.

### Changing the recipient

Edit the **Email Article for Review** node's **Send To** field. Make sure the selected Gmail credential is authorized to send mail.

### Changing the model or token limit

Edit the **Anthropic Chat Model** node. The current model identifier is `claude-sonnet-5-5`, and `maxTokensToSample` is `16000`. Confirm model availability and provider limits before changing these settings.

### Changing the article prompt or categories

- Edit the system message in **Write Article** to change writing requirements or output structure.
- Edit the category list in **Pick Category** to change the rotation.
- Keep the output headings and formatting consistent with the expressions in **Extract Metadata**, which parse the model response.

## Run and operate the workflow

The workflow is designed to run on its schedule. For development and verification, use n8n's manual execution feature.

After a successful run, verify:

- The **Write Article** node returned the expected article structure.
- **Extract Metadata** populated the title, topic, category, date, article, metadata, and review fields.
- **Convert to HTML** produced the `html` field.
- **Email Article for Review** completed successfully.
- **Save to History** inserted the expected record.

The workflow sends an email for review. It does not publish the article to Medium automatically.

## Article output

The prompt asks the model to return:

- Article metadata: title, category, topic, difficulty, estimated reading time, five tags, and quality status.
- A paste-ready Markdown article with sections in a specified order.
- A quality review covering technical accuracy, code quality, structure, originality, and references.
- Known limitations or items requiring verification.
- Editorial notes for the reviewer.

The Markdown-to-HTML node converts the generated article together with metadata and review information into the email body. Review notes are included in the email output and are not intended to be pasted into the Medium article.

## Data stored in article_history

After the Gmail node succeeds, **Save to History** inserts:

| Field | Value source |
| --- | --- |
| `category` | Selected category |
| `title` | Extracted article title |
| `topic` | Extracted topic |
| `published_date` | Workflow date in `yyyy-MM-dd` format |

The table is also read at the start of the next run to help choose a category and avoid repeating titles within the selected category.

## Security and privacy

Treat exported n8n workflow JSON as configuration that must be reviewed before sharing.

- The workflow JSON includes the configured email recipient and credential references/names. Review these before committing the file to a repository.
- Normal workflow exports do not contain the secret values stored in n8n Credentials, but hardcoded passwords, API keys, authorization headers, or tokens placed directly in node parameters can appear in the JSON.
- Never commit decrypted credential exports, API keys, OAuth tokens, passwords, private keys, or n8n instance encryption keys.
- Store credentials in n8n's credential manager and restrict access to the n8n instance.
- Prefer a private GitHub repository if the workflow contains personal recipient details or internal configuration.
- If a secret is accidentally committed, revoke or rotate it. Removing it in a later commit does not erase it from Git history.
- Keep generated articles, execution logs, database backups, and n8n instance data out of source control unless there is a deliberate, reviewed reason to version them.

A workflow JSON file is not a complete n8n instance backup. Back up the n8n database and the instance encryption key securely using a separate backup process.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| Workflow does not run at 7:00 AM | Check that the workflow is active, the Schedule Trigger is configured, and the workflow/instance timezone is correct. |
| Anthropic node fails | Check the selected credential, provider access, model availability, and provider limits. |
| Gmail node fails | Check the selected Gmail OAuth2 credential, authorization, and recipient address. |
| Data Table node fails | Confirm that `article_history` exists and has the expected column names and types. |
| Category or duplicate-topic logic behaves unexpectedly | Inspect the rows returned by **Get Past Articles** and the expressions in **Pick Category** and **Write Article**. |
| Title or article fields are empty | Inspect the raw model response and ensure it follows the required output format used by **Extract Metadata**. |
| Email formatting is incorrect | Inspect the `html` output from **Convert to HTML** and the email node's message field. |
| History row is missing | Check whether the Gmail node succeeded; the history node runs after email delivery in this workflow. |

## Limitations and notes

- **Timezone is external configuration.** The JSON sets the trigger hour to 7 but does not embed `Asia/Kolkata`.
- **Model dependency.** This workflow uses an Anthropic chat model, not a local Ollama model. Provider access and model availability are required.
- **Category rotation is count-based.** The selected category is based on the total number of history rows with a non-empty title modulo ten. It does not independently balance categories or select the least-used category.
- **History-based duplicate avoidance is title-based.** The prompt receives previous titles for the selected category and asks the model not to repeat or closely overlap them. This is prompt guidance, not a strict uniqueness constraint enforced by the database.
- **History is saved after email.** If email delivery fails, the history insertion node is not reached. If email succeeds but the history insertion fails, an email may be sent without a corresponding history row.
- **The `published_date` field is a generation date.** The workflow does not publish to Medium or confirm publication.
- **No automated tests are included in the uploaded workflow JSON.** Validate changes through manual executions and inspection of node outputs.
- **No screenshots or demo media were included with the supplied workflow file.**

## Contributing

Contributions and improvements are welcome.

1. Fork the repository or create a feature branch.
2. Make a focused change to the workflow or documentation.
3. Import the modified workflow into a test n8n instance.
4. Test the affected nodes and inspect execution output.
5. Ensure credentials and secrets are not included in exported files.
6. Submit a pull request describing the change and how it was tested.

Do not include live credentials, tokens, passwords, personal execution data, or generated private content in pull requests.

## License

**License: Not specified.**

No license information was present in the supplied workflow JSON. Add a `LICENSE` file and update this section if you intend to grant others permission to use, modify, and distribute this project. Without an explicit license, others should not assume that the repository is open for reuse.
