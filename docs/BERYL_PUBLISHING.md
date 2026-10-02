# Publishing PROWL to `beryl`

The private repository `peace-amani/prowl-bot` contains the source. GitHub Actions builds an obfuscated copy and force-pushes that copy to the public repository `peace-amani/beryl`.

## 1. Create a fine-grained token

Create a fine-grained personal access token at:

<https://github.com/settings/personal-access-tokens/new>

Use these settings:

- **Resource owner:** `peace-amani`
- **Repository access:** Only select repositories → `beryl`
- **Repository permissions:** Contents → **Read and write**
- Leave all other repository permissions at their defaults.

Copy the token once. Do not commit it or paste it into workflow YAML.

## 2. Add the token to the private source repository

Open:

<https://github.com/peace-amani/prowl-bot/settings/secrets/actions/new>

Create this Actions secret:

- **Name:** `BERYL_REPO_TOKEN`
- **Secret:** the fine-grained token from step 1

The workflow uses this secret only during the publish step. The token is never written into the generated release files.

## 3. Run the workflow

The workflow is located at `.github/workflows/publish-obfuscated.yml` and runs when:

- A commit is pushed to `main`; or
- You manually select **Actions → Publish obfuscated distribution → Run workflow**.

The workflow:

1. Checks out the private source.
2. Installs only `javascript-obfuscator` with npm scripts disabled.
3. Validates the entrypoint syntax.
4. Copies tracked files and obfuscates JavaScript files.
5. Rewrites release links from the private `prowl-bot` repository to public `beryl`.
6. Verifies that `.env`, session state, database state, and bot settings are absent.
7. Force-pushes the generated distribution to `peace-amani/beryl`.

The force-push is intentional: `beryl/main` is a generated distribution branch, not a development branch. Do not make manual source edits in `beryl` because the next successful publication replaces them.

## 4. Verify the first run

After adding the secret, run the workflow manually once and check:

- The **Publish obfuscated distribution** job is green in the private repository.
- `peace-amani/beryl` contains the generated files.
- No `.env`, `session/`, `auth_info/`, `data/`, `.session_id_hash`, or `bot_name.json` files were published.
- The public repository does not expose the `BERYL_REPO_TOKEN` value.

If the publish step fails with a 403, confirm that the token is fine-grained, is restricted to `beryl`, has **Contents: Read and write**, and was saved as an Actions secret on `prowl-bot` with the exact name `BERYL_REPO_TOKEN`.
