# Kozocom Marketplace

Public Claude Code marketplace for reusable plugins and skills.

## Install this marketplace and plugin next-seo-skills from this marketplace

```bash
claude plugin marketplace add doanhv-kozocom/kozocom-marketplaces
claude plugin install next-seo-skills@kozocom-marketplace
```

## Available plugins

### `next-seo-skills`

SEO workflows for Next.js and web projects.

Included skills:

- `seo-plan`
- `seo-audit`
- `seo-implement`
- `seo-fix`

## Use a plugin skill

After installing a plugin, run its skills with the plugin namespace.

Example:

```text
/seo-plan
/seo-implement
/seo-fix
/seo-audit https://example.com
```

## Repository structure

```text
.claude-plugin/
  marketplace.json
plugins/
  next-seo-skills/
    .claude-plugin/
      plugin.json
    skills/
      seo-plan/
      seo-audit/
      seo-implement/
      seo-fix/
```

## License

MIT
