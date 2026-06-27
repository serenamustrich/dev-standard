---
name: company-dev-standards
description: Apply Western Post/ACMP company development standards for backend Java, Vue frontend, MySQL schema, data dictionary, supplier workflows, branch comparisons, database diffs, documentation updates, or reviews where code and database consistency must follow company conventions.
---

# Company Dev Standards

Use this skill when reviewing, implementing, or documenting company-style changes for ACMP or adjacent Western Post systems.

## Workflow

1. Identify the source of truth before changing anything:
   - current local code and database,
   - test or develop branch code,
   - test/prod database schema,
   - confirmed business requirements.

2. Read `references/company-standards.md` when the task touches backend structure, frontend field binding, database schema, data dictionary, supplier workflow, external integrations, or documentation.

3. Separate three concepts in the answer or implementation:
   - company coding/database convention,
   - current confirmed business rule,
   - accidental drift or legacy design.

4. Never let convention silently override confirmed business rules. If they conflict, preserve the business rule and call out the required alignment work.

5. For database work, compare actual schemas when credentials are available:
   - table list,
   - columns,
   - indexes,
   - `SHOW CREATE TABLE` or `mysqldump --no-data`.

6. For code work, update all layers together:
   - database SQL,
   - Entity and `@TableField`,
   - Mapper and query fields,
   - DTO/VO/Wrapper,
   - Service validation,
   - frontend API types,
   - frontend page bindings,
   - docs and requirement systems when requested.

## Key Guardrails

- Do not use compatibility shims when the user asks to keep frontend, backend, and database consistent.
- Do not remove local user changes or dirty worktree changes unless explicitly requested.
- Do not treat build artifacts, caches, or `dist` output as meaningful business diffs.
- Require migration SQL or DDL documentation for database changes.
- Keep supplier draft behavior, OA submission behavior, external sync behavior, and data dictionary structure aligned with confirmed requirements.
- For supplier OA integration, treat GitLab `develop-test` developer changes as the company-compliant baseline unless the user names a newer source of truth.
- Keep OA OpenAPI settings, OA datasource settings, and OA workflow table names centralized in config such as `PlatformOAConfig`; do not hardcode OA table names, test file URLs, test users, or workflow IDs inside services.
- For creator identity, use the framework-provided OA code capability such as `AuthUtil.getOACode()`; do not hardcode work numbers or silently fall back to fake/default OA users.
- Distinguish supplier creator from applicant: creator comes from the logged-in account, applicant comes from the manually entered work number and drives the OA approval flow.
- Store OA workflow ID/requestId separately from OA document code; display and persist the OA document code when available, query it from the configured OA main table, and keep a retry/manual-fetch path when it is not immediately available.
- Mark OA push failures as push failures instead of approval-in-progress when no OA workflow ID is returned; allow retry only for failed records without an OA workflow ID.
- Push real uploaded qualification attachments to OA using file metadata/content from the file service; never ship fixed test attachments or fixed MinIO URLs as production logic.
- Use enums for supplier, qualification, and OA push states; store stable codes and convert to Chinese descriptions only at display or external-payload boundaries.
- Physical database table and field names must not use MySQL keywords or bare generic names such as `status`, `comment`, `type`, `name`, `code`, `sort`, `order`, `group`, `key`, or `value`; use business-prefixed fields such as `contract_status`, `dict_code`, and `borrow_remark`, and map Java properties with `@TableField` when API names stay generic.
- For MySQL projects, validate physical table and column names against the current MySQL server's `INFORMATION_SCHEMA.KEYWORDS`; treat both reserved and non-reserved keyword rows as forbidden physical identifiers under company standards.
- Physical primary-key columns must use integer numeric types such as `int` or `bigint`. Business identifiers such as `seq_type`, `dict_code`, `supplier_code`, `order_no`, or other string codes must not be primary keys; keep them as normal business fields with unique indexes when uniqueness is required.

## Common Outputs

- A schema comparison report with exact field and index differences.
- A company standards document for developers.
- A code review finding list focused on inconsistency risks.
- A scoped implementation plan that lists every layer to change.
- Updated backend/frontend/database/requirement documentation.
