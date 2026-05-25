---
name: zeabur-template
description: Create, validate, update, and maintain Zeabur templates using official Template Resource YAML format and maintainer best practices.
---

# Zeabur Template Skill

Use this skill when the user asks to create a new Zeabur template, update an existing template, or maintain template quality over time.

## Sources of truth

- https://zeabur.com/docs/en-US/template
- https://zeabur.com/docs/en-US/template/template-catalog
- https://zeabur.com/docs/en-US/template/template-format
- https://zeabur.com/docs/en-US/template/maintain-template

## What this skill does

- Creates a new template YAML in Zeabur Template Resource format.
- Refactors existing template YAML while preserving deployability.
- Prepares safe version bumps (image tag/env/service config updates).
- Runs a maintainer checklist before publish/update.
- Provides exact CLI commands for deploy and update flows.

## Workflow

1. Confirm task type

- `create`: new template from scratch.
- `update`: modify an existing published template.
- `maintain`: audit and improve template quality without major behavior changes.

2. Gather required inputs

- Template name, description, icon URL, tags.
- Services list and startup dependencies.
- Runtime type per service: `PREBUILT` or `GIT`.
- Variables to expose to users (`STRING` or `DOMAIN`).
- Localization languages needed (for example `zh-TW`, `ja-JP`).
- For update tasks: template code from URL, e.g. `71HORL`.

3. Edit YAML using official structure

Required top-level structure:

```yaml
apiVersion: zeabur.com/v1
kind: Template
metadata:
  name: <TemplateName>
spec:
  description: <Short description>
  icon: <Icon URL>
  tags: []
  readme: |-
    # <TemplateName>
    <Usage docs>
  variables: []
  services: []
localization: {}
```

Implementation rules:

- Keep `apiVersion: zeabur.com/v1` and `kind: Template` unchanged.
- Use explicit image tags, avoid `latest` when possible.
- Use `${PASSWORD}` or other generated placeholders for secrets, never hardcode credentials.
- Use `domainKey` only when binding a `DOMAIN` variable to a service.
- Add `dependencies` when startup order matters.
- Keep variable `key` names consistent with service env references.

4. Validate operationally

Before submission/update, run a real deploy verification:

```bash
npx zeabur@latest template deploy -f <template-file>.yml
```

For published template update:

```bash
npx zeabur@latest template update -c <template-code> -f <template-file>.yml
```

5. Final maintainer checklist

- Deployability: template deploys and boots correctly.
- Metadata quality: name/description/readme are complete and clear.
- Taxonomy quality: icon and tags are appropriate.
- Variable UX: all user-facing variables include clear `name` and `description`.
- Security: no hardcoded secret values.
- Reproducibility: version tags are specific and intentional.
- Compatibility: existing deployed projects are not silently broken by update assumptions.

## Update strategy

When updating a published template:

1. Read current YAML and identify behavioral changes.
2. Prefer minimal diffs for each release.
3. Separate high-risk changes (major image bump, env contract changes) from low-risk cleanup.
4. Document migration/redeploy notes in `readme` when user action is needed.

## Output contract

When this skill is used, return:

1. Changed files list.
2. Final YAML diff summary (service/version/env/dependency changes).
3. Exact commands to verify and publish/update.
4. Risks and rollback suggestion if changes are not backward compatible.

## Anti-patterns to avoid

- Massive refactor mixed with version bump in one step.
- Using `latest` for production-critical services.
- Missing `readme` or thin one-line readme.
- Inconsistent variable keys between `spec.variables` and service env.
- Hardcoded tokens/passwords in YAML.
