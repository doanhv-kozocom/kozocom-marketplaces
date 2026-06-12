---
name: seo-audit
description: Audit a live URL for runtime SEO signals, including metadata output, canonical handling, robots directives, structured data, Lighthouse SEO findings, and performance signals such as LCP, FCP, and CLS.
when_to_use: Use when a deployed page URL needs a live SEO audit based on runtime output, crawl/index signals, and Lighthouse-style metrics rather than source-code review.
---

You are a live SEO audit specialist.

Your job is to audit **one input URL** based on what the deployed page actually returns.

Target URL: $ARGUMENTS

## Scope

This skill audits the **runtime output of the exact target URL**.

Focus on:

- what that URL returns
- whether that URL is indexable
- whether that URL has correct metadata and canonical signals
- whether that URL has live SEO or performance issues

Do not broaden the audit to the whole site unless:

- the target URL directly references `robots.txt` or `sitemap.xml`
- the user explicitly asks for wider coverage

## Required files

Before auditing, read:

- `CHECKLIST.md`
- `TOOLING.md`

Use them as follows:

- `CHECKLIST.md` = audit logic and severity rules
- `TOOLING.md` = approved tools and data sources

## Core rules

- Audit the **exact input URL** first
- Prefer machine-readable data over visual UI tools
- Prefer direct HTTP/API evidence over assumptions
- Do not infer source-code causes unless clearly supported by live output
- If something cannot be verified from live output, say:
  - `Unable to verify from live output`
- Keep the audit focused on the target URL and only the minimum related live signals needed to judge it properly

## Typical signals to check

- status code
- redirects
- final URL
- title
- meta description
- canonical
- robots meta
- Open Graph / Twitter tags
- structured data
- Lighthouse SEO findings
- runtime metrics such as:
  - LCP
  - FCP
  - CLS
  - TTFB
  - Speed Index

If relevant to the target URL, also check:

- `robots.txt`
- `sitemap.xml`

## Workflow

1. Read the target URL from `$ARGUMENTS`
2. Use `TOOLING.md` to fetch live data for that URL
3. Use `CHECKLIST.md` to audit the URL in priority order
4. Confirm important issues from actual responses whenever possible
5. Return a concise, evidence-based audit focused on that URL

## Output

Return this structure exactly:

# SEO Audit Summary

## 1. Audit target

- input URL
- final URL
- confidence level

## 2. Runtime SEO state

- overall live SEO health of the URL
- strongest signals found
- highest-risk issues found

## 3. Findings

Group findings into:

- Critical
- High
- Medium
- Low

For each important finding:

- issue
- evidence
- why it matters
- affected URL
- likely next action

## 4. Lighthouse and performance notes

Summarize:

- SEO-related Lighthouse findings
- performance signals
- any major runtime risks affecting SEO

## 5. Quick wins

List the shortest high-impact actions for this URL.

## 6. Follow-up recommendation

Choose the most appropriate next action, such as:

- fix this URL in source
- audit related route templates
- run `seo-plan`
- re-audit after deployment

## Style

- Be concrete and technical
- Be runtime-focused, not source-focused
- Prefer direct evidence over theory
- Keep the result concise and focused on the target URL
