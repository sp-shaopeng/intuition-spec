# Publication notes

This is a documentation-only public edition of the InTuition specifications. It describes business requirements, user journeys, rules and technical design in prose and high-level sequence diagrams. It is not an application, deployment package or operational evidence archive.

The source is the current private working copy, including the September requirements corrections and additional nonignored specification documents. The [current requirements register](../specs/CURRENT-REQUIREMENTS.md) is the authority when it conflicts with an older plan or historical review. Relative specification paths are retained so decisions can be traced across documents.

The initial export includes every non-evidence Markdown specification discovered in the tracked and nonignored working-copy file lists at that snapshot. It also includes the approved deployment plan as a supporting decision record; that exception is prose-only. The [inventory](publication-inventory.md) lists every included source path and its source category, plus the later reading guides and two HTML-to-Markdown conversions. The [specification index](specification-index.md) provides readable links. Later private repair drafts do not silently replace the published requirements.

Credential values, example and test passwords, private account identifiers, personal contact details, private repository addresses, deployment hosts, machine addresses and local key paths are withheld. A redaction marker records that a value was intentionally withheld. Secret variable names and general security requirements may remain where they explain behavior. Email addresses, including illustrative and test addresses, are withheld; non-contact example domains may remain.

Fenced implementation examples, commands and configuration are replaced as whole examples with neutral omission notices rather than guesses about their behavior. Genuine behavioral scenarios, useful information fields and readable lifecycle transitions are retained in prose where they can be expressed without implementation syntax. The surrounding requirements remain available. These explanations are not executable instructions or a replacement for the private implementation.

The later diagram revision allows Mermaid sequence notation inside Markdown. GitHub renders it as documentation diagrams; it contains no application, test or deployment code. The earlier zero-code-block checks describe their dated publications, before these diagrams were added. New checks allow only reviewed Mermaid sequence blocks and continue to exclude HTML and other fenced code. Each diagram states whether it describes requirements, the written workflow or a proposal. A rendered diagram does not prove that the application behaves that way. The [diagram guide](sequence-diagrams.md) maps the visual summaries to the detailed text records.

Application code, test code, infrastructure configuration, repository history, screenshots, logs, deployment state, environment files and raw evidence are excluded. Links to excluded implementation or evidence material point here. They describe private supporting material, not an omitted public download.

This snapshot does not change the private source, revoke credentials, or demonstrate that an older exposed credential has been invalidated. GitHub explains that deleting a secret from a current file does not remove it from prior history and recommends revocation or rotation for exposed credentials. This edition starts without that history. See [GitHub guidance on removing sensitive data](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository).

Historical test results and reviews are retained as reported requirements context. They are not claims that the public documentation was executed, deployed or independently verified on the publication date.

## History and review roles

The founder used AI assistants with role names such as business analyst, developer, QA and “Boss”. Names and sign-offs in historical documents refer to those AI-assisted sessions, not to employees, customers or independent human certification. Review reports record what was claimed at the time; they do not establish a fresh passing result.

The public guides describe the inspected October 2026 state. Older specifications preserve decision history, and the current requirements register settles later corrections. When a linked private screenshot, test report or implementation file is excluded, its link is labelled as a publication note rather than evidence the reader can inspect.
