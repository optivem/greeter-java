# Pipeline Security

Rules for designing secure CI/CD pipelines.

## Secrets Handling

- **Never pass secrets via action inputs (`with`)**. Action inputs are interpolated directly into YAML expressions and may appear in logs. Always pass secrets through environment variables (`env`), which GitHub automatically masks in logs.

- **Never use `${{ }}` interpolation with secret values in `run` blocks**. Direct interpolation substitutes the raw value into the shell script, enabling script injection if the secret contains special characters. Use environment variables instead:

  ```yaml
  # Bad - direct interpolation, vulnerable to script injection
  run: echo "${{ secrets.MY_SECRET }}"

  # Good - environment variable, masked and safe
  run: echo "$MY_SECRET"
  env:
    MY_SECRET: ${{ secrets.MY_SECRET }}
  ```

- **Validate secrets upfront**. Check that all required secrets and variables are defined at the start of the pipeline, before any real work begins. Use a dedicated validation step to fail fast with clear error messages.

- **Never echo or log secret values**. Only validate that secrets are non-empty, never print their contents.

## Action Design

- **Composite actions should read secrets from environment variables, not inputs**. The `inputs` context in composite actions is interpolated into `run` blocks via `${{ inputs.* }}`, which has the same script injection risk as `${{ secrets.* }}`.

- **Prefer `env` over `with` for any value that may contain untrusted or sensitive content**. This applies to secrets, user-provided inputs, and any value that could contain special characters.

## Permissions

- **Use least-privilege permissions**. Declare only the permissions each job actually needs. Never use blanket `permissions: write-all`.

- **Scope tokens to the minimum required access**. Use fine-grained personal access tokens or GitHub App tokens instead of broad-scope tokens where possible.

## Third-Party Actions

- **Pin third-party actions to a specific version or SHA**. Avoid using `@main` or `@latest` for third-party actions, as upstream changes could introduce malicious code.

- **Audit third-party actions before use**. Review the action source code and ensure it comes from a trusted source.
