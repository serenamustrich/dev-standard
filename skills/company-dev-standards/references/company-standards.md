# Company Development Standards Reference

## 1. Backend Structure

Use the company-style backend package layout:

```text
com.wpglb.scm
├── aspect
│   └── annotation
├── config
├── controller
│   └── openapi
├── enums
├── exception
├── mapper
├── pojo
│   ├── dto
│   ├── entity
│   └── vo
├── service
│   └── impl
├── utils
└── wrapper
```

Responsibilities:

- `controller`: internal/admin APIs.
- `controller/openapi`: external callbacks and open APIs.
- `pojo/entity`: database entities.
- `pojo/dto`: request and edit objects.
- `pojo/vo`: response objects.
- `wrapper`: entity-to-VO conversion.
- `enums`: states and stable code enums.
- `aspect`: auth, logging, or other cross-cutting logic.
- `config`: external platform configuration.
- `utils`: narrow reusable helpers.

## 2. Java Naming

Entity names should preserve table prefixes:

| Table | Entity |
| --- | --- |
| `scm_supplier` | `ScmSupplier` |
| `scm_supplier_bank_account` | `ScmSupplierBankAccount` |
| `scm_supplier_cert` | `ScmSupplierCertification` |
| `scm_dict` | `ScmDict` |
| `scm_dict_type` | `ScmDictType` |
| `scm_dict_group` | `ScmDictGroup` |
| `ts_bank_cnaps` | `TsBankCnaps` |
| `scm_flow_log` | `ScmFlowLog` |

Use matching names for Mapper, Service, DTO, VO, and Wrapper.

## 3. Database Table Names

- `scm_`: SCM/ACMP business tables.
- `ts_`: external master data or synchronized base data.
- Use lowercase snake case.
- Do not use unclear abbreviations.

## 4. Database Field Names

The company test schema favors object-prefixed field names:

| Table | Company-style fields |
| --- | --- |
| `scm_dict` | `dict_code`, `dict_name`, `dict_sort`, `dict_status` |
| `scm_dict_group` | `group_code`, `group_name`, `group_sort`, `group_status` |
| `scm_dict_type` | `type_category`, `type_sort`, `type_status` |
| `scm_dict_reference_rule` | `rule_target_table`, `rule_match_mode` |
| `ts_country` | `country_code`, `country_name_cn`, `country_name_en` |

Hard rules:

- Do not use SQL keywords or common reserved words as physical database column names, even when MySQL allows them with backticks.
- Do not create bare generic columns such as `status`, `type`, `name`, `code`, `sort`, `comment`, `order`, `group`, `key`, `value`, `rank`, `desc`, `condition`, `level`, `index`, or `default`.
- Use business-object-prefixed names instead, for example `contract_status`, `asset_status`, `payment_apply_status`, `dict_code`, `dict_name`, `dict_sort`, `borrow_remark`.
- Existing API/DTO/VO property names may keep clear business semantics such as `status` for request filters or response fields, but Entity fields must map to compliant physical columns with `@TableField`.
- When a field rename is required, update the DDL, local database schema, Entity mapping, Mapper SQL, Service update/query wrappers, frontend API types if affected, and related documents together.
- For MySQL 8 projects, the verification source is the live server table `INFORMATION_SCHEMA.KEYWORDS`, not a manually maintained keyword list. Join current schema table and column identifiers against that table and treat every keyword row, including non-reserved keywords, as forbidden for physical database identifiers.

## 5. MyBatis-Plus Mapping

- Java fields use camel case.
- Database fields use snake case.
- Use default camel-to-snake mapping only when names match.
- Use `@TableField` whenever the database field differs from default mapping.
- After a field rename, check Entity, DTO, VO, Mapper SQL, Wrapper, Service, frontend API types, frontend page bindings, and docs.

## 6. Primary Keys

Use `IdType.ASSIGN_ID` for system-owned business tables and logs.

Physical primary-key columns must use integer numeric types such as `int` or `bigint`.

Do not use business strings, codes, names, or enum-like fields as primary keys. Examples that must not be primary keys include:

- `seq_type`
- `dict_code`
- `supplier_code`
- `order_no`
- `contract_no`
- any other varchar/char/text business identifier

When a business identifier must be unique, keep the numeric primary key and add a separate unique index on the business field.

For externally synchronized master data:

- preserve external stable IDs when the external system provides them;
- otherwise generate an internal ID and add unique constraints on stable external business codes.

## 7. Time Fields

Typical SCM business tables use:

- `created_time`
- `updated_time`

External master-data or legacy-style tables may use:

- `create_time`
- `update_time`

Creation and update times should be non-null. Update time should be maintained by backend logic, not by arbitrary frontend input.

## 8. Logical Delete

Use `is_deleted` for business tables:

- `0`: active.
- `1`: deleted.

When unique values should be reusable after logical deletion, use generated active columns or composite unique indexes.

## 9. Indexes and Constraints

Do not rely only on frontend or backend duplicate checks. Use database constraints for critical uniqueness.

Important ACMP constraints:

- dictionary type code unique;
- dictionary group code unique within a type;
- dictionary item code unique within a type;
- supplier code unique;
- supplier Chinese name unique within country or region;
- Chinese mainland unified social credit code unique;
- non-mainland tax number or TIN/VAT unique according to business rule;
- OA process ID unique;
- active bank account unique;
- one default bank account per supplier;
- supplier and branch-company relation unique;
- supplier and supplier-type relation unique.

## 10. Data Dictionary

Use a three-layer dictionary model:

```text
dictionary type
  └── group within type
        └── dictionary item
```

Tables:

- `scm_dict_type`
- `scm_dict_group`
- `scm_dict`
- `scm_dict_reference_rule`

Dictionary items may need an OA mapping value such as `dict_oa_value`.

Dictionary codes must be stable and should not change when display names change.

## 11. Attachments

Company-style attachment fields include:

- `file_code`
- `file_name`
- `file_ext`
- `file_size`
- `file_url`

Prefer `file_code` as the durable reference. Do not use direct URL as the only recoverable value.

## 12. JSON Storage

The company test schema often uses `text` for JSON-like data; local ACMP may use MySQL `json`.

Tradeoff:

- `text`: more compatible, but no database JSON validation.
- `json`: validates format and supports JSON querying, but requires valid JSON.

If using `text`, validate JSON in backend before writing. Do not store only opaque JSON for important recoverable business data; keep key fields structured.

## 13. External APIs and Logs

Company-style external API structure:

- `controller/openapi`
- auth annotation such as `AuthOpenApiRequest`
- auth aspect such as `OpenApiRequestAuthAspect`
- platform config such as `PlatformOAConfig`
- external push services such as `SupplierOaPushService`
- request log table such as `scm_flow_log`

For supplier external integration, distinguish:

- raw OpenAPI request logs,
- business integration logs for OA/FMS/NS sync status.

`scm_flow_log` and `scm_supplier_integration_log` are not automatic substitutes for each other.

## 14. Supplier OA Integration

For supplier OA integration, use the developer-updated GitLab `develop-test` branch as the company-compliant baseline unless the user identifies a newer source of truth.

Configuration and hardcoding:

- Keep OA OpenAPI settings, OA datasource settings, workflow main-table names, and detail-table names in `PlatformOAConfig` and environment configuration.
- Do not hardcode OA workflow table names, detail-table names, JDBC drivers, fixed test attachment URLs, fixed file names, test work numbers, fake OA user IDs, workflow IDs, or request IDs in business services.
- Sensitive values such as passwords, client secrets, and signing keys may exist in environment configuration but must not be copied into reports, logs, requirement text, or final answers.
- SQL table-name interpolation is acceptable only when the table name comes from controlled application configuration, never from frontend input or external requests.

Creator and applicant:

- Supplier creator and applicant are separate business concepts.
- Creator comes from the currently logged-in account and should use the framework-provided OA code capability, such as `AuthUtil.getOACode()`.
- Do not infer creator work numbers from email, display name, or ad hoc fields unless an officially confirmed mapping is unavailable and the user explicitly approves the fallback.
- Do not hardcode creator work numbers or silently fall back to `0` or another fake OA user.
- Applicant comes from the manually entered work number; the backend queries OA user and department information from that work number, and the OA approval flow follows the applicant configuration.

OA push status:

- If OA returns a workflow ID/requestId, store it and treat the supplier as approval-in-progress.
- If OA does not return a workflow ID/requestId or the push fails, mark the record as OA push failed instead of approval-in-progress.
- Retry OA push only for failed records that do not already have a workflow ID/requestId.
- Once a workflow ID/requestId exists, do not create another OA workflow for the same supplier submission.

OA process code:

- Store the OA workflow ID/requestId separately from the OA document code.
- The UI should display the OA document code when available, not the workflow ID/requestId.
- Query the document code from the configured OA main table. The current supplier-access flow uses `djbm` as the OA document-code field.
- After successful OA push, retry document-code lookup asynchronously. The accepted local pattern is 5 attempts with a 3-second interval.
- If the code is still unavailable, keep the workflow ID/requestId, allow a manual fetch action, and persist the code when it becomes available.

Attachments and state values:

- Push real uploaded qualification attachments to OA using stored file metadata and file-service content.
- Do not send fixed test files, fixed MinIO URLs, or placeholder attachment content in production logic.
- Use enums for supplier, qualification, and OA push states. Store stable enum codes and translate to Chinese descriptions only for display or external payloads.

One-off SQL:

- Structural database changes still need formal DDL, migration SQL, or release-record evidence.
- Temporary data-fix SQL does not need to remain as permanent business code, but it must be traceable in delivery or database-change records.

## 15. Frontend Standards

- APIs live under `apps/web-antd/src/api/*`.
- Views live under `apps/web-antd/src/views/*`.
- System configuration pages live under `views/system/*`.
- Supplier pages live under `views/supplier/*`.
- Use stable API types; keep frontend fields aligned with backend DTO/VO.
- Use `available` for generic uniqueness-check responses when possible.
- Frontend validation improves UX but backend and database must still enforce correctness.

## 16. ACMP Business Boundaries

Do not override these confirmed business decisions with generic convention:

- Draft supplier data may be incomplete before OA submission.
- Draft behavior can require separate draft tables.
- OA submission must trigger frontend and backend required-field validation.
- Approved suppliers sync to external systems.
- Supplier Chinese name is unique within country or region.
- Country data comes from OA or external synchronization and is not a normal dictionary.
- Bank data comes from external synchronized master data.
- Branch company/signing entity participates through data dictionary style configuration, not as an independent menu object.
- Data dictionary must support future type, group, and item extension.

## 17. Review Checklist

Before finalizing a change:

1. Compare actual schemas if database credentials exist.
2. Ignore build artifacts and caches in code diffs.
3. Confirm table and field names match Entity mappings.
4. Confirm DTO, VO, Wrapper, Mapper, Service, frontend API, and page bindings are updated together.
5. Confirm required unique indexes exist.
6. Confirm migration SQL is committed or documented.
7. Confirm historical data migration is planned.
8. Confirm supplier draft, OA submission, callback, and external sync paths still work.
9. Update backend, frontend, database, API, and requirement docs when requested.
10. Update TAPD or requirement-confirmation systems only from final business wording, not process notes.
11. Confirm there are no hardcoded OA workflow tables, test attachments, test users, or fake workflow IDs.
12. Confirm creator identity uses the framework OA-code capability and applicant identity still follows the manually entered work number.
13. Confirm OA workflow ID/requestId and OA document code are stored and displayed according to their separate meanings.
