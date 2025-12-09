<!--
  SYNC IMPACT REPORT
  ==================
  Version change: 1.0.0 → 1.1.0

  Modified principles:
    - I. Accuracy: Added specific reconciliation rules
    - II. Consistency: Defined explicit formats (ISO 8601, decimal, enums)
    - III. Traceability: Defined critical actions list
    - IV. Security: Expanded to full security standards
    - V. User-Friendly: Added measurable UX metrics (clicks, error messages)
    - VI. Testable Standards: No change

  Added sections:
    - Security Standards (expanded from principle)
    - Module-Specific Rules (Product, Purchase, Production, Sales, Accounting, Reporting)
    - Performance Standards (tiered API response times)
    - Testing Requirements (coverage, regression, load testing)
    - Workflow Compliance Gates (state machines, approval thresholds, code review)

  Removed sections: None

  Templates requiring updates:
    - .specify/templates/plan-template.md: ✅ Compatible (Constitution Check section aligns)
    - .specify/templates/spec-template.md: ✅ Compatible (Requirements/Success Criteria align)
    - .specify/templates/tasks-template.md: ✅ Compatible (Phase structure aligns)

  Follow-up TODOs: None
-->

# IHS ERP Constitution

*Global rules and quality standards for all modules, features, and tasks*

---

## Core Principles

### I. Accuracy

All transactions, reports, and stock data MUST be mathematically correct and verifiable.

**Testable Requirements**:
- Sum of line items MUST equal document total (tolerance: ±0.00 for quantities, ±0.01 for currency rounding)
- Inventory balance = Opening + Purchases - Sales - Adjustments (verified by automated reconciliation)
- Financial reports MUST tie to general ledger within ±0.01

**Rationale**: An ERP system's core value lies in providing reliable data for business decisions. Inaccurate data leads to financial losses, compliance violations, and eroded trust.

### II. Consistency

All modules (Accounts, Sales, Inventory, Reports, Settings) MUST follow identical data formats and behavior patterns.

**Testable Requirements**:
- Dates: ISO 8601 format (`YYYY-MM-DD` or `YYYY-MM-DDTHH:mm:ssZ`)
- Currency: `decimal(15,2)` with 2 decimal places displayed
- Quantities: Integer ≥0 (no negative stock allowed)
- IDs: UUID v4 format for all entity identifiers
- Status fields: Standardized enums defined per module (no free-text status)

**Rationale**: Consistent behavior across modules reduces training time, minimizes user errors, and ensures predictable system behavior.

### III. Traceability

Every critical action MUST be logged in immutable audit logs with complete context.

**Critical Actions (MUST be logged)**:
- CREATE/UPDATE/DELETE on: financial records, inventory adjustments, user accounts, permissions
- Price changes, discount applications, approval decisions
- Login attempts (success and failure), session events
- Report generation, data exports

**Log Entry Requirements**:
- User ID (who)
- Timestamp (when, UTC)
- Module and entity (where)
- Action type (what)
- Before/after values for updates (change detail)
- IP address and session ID (context)

**Retention**: Minimum 7 years; logs MUST be immutable after creation.

**Rationale**: Traceability is essential for compliance, debugging, security investigations, and accountability.

### IV. Security

All data and actions MUST be protected through defense-in-depth security controls.

**Testable Requirements**: See [Security Standards](#security-standards) section.

**Rationale**: ERP systems contain sensitive financial and business data. Unauthorized access or actions can lead to fraud, data breaches, and regulatory penalties.

### V. User-Friendly

Interfaces MUST be efficient and self-explanatory for both admins and end-users.

**Testable Requirements**:
- Primary actions MUST complete in ≤5 clicks from dashboard
- Error messages MUST specify: field name + expected format + example
- Loading states MUST appear within 200ms of user action
- All workflows MUST have accessible help documentation
- Form validation MUST occur on blur (immediate feedback)

**Rationale**: Usability directly impacts adoption and productivity. Complex interfaces lead to errors, resistance to adoption, and increased support burden.

### VI. Testable Standards

Every rule in this Constitution MUST be measurable and automatically verifiable where possible.

**Testable Requirements**:
- All quality metrics MUST have numeric thresholds
- All constraints MUST have enforcement mechanisms defined
- All success criteria MUST be verifiable via automated tests or documented manual procedures

**Rationale**: Vague standards cannot be verified or enforced. Measurable criteria enable automated testing, objective code reviews, and clear acceptance decisions.

---

## Quality Standards

- **Data Validation**: All forms MUST enforce:
  - Currency: `decimal(15,2)`
  - Quantities: `integer ≥0`
  - Dates: ISO 8601
  - IDs: UUID v4
  - Required fields: Cannot submit with empty mandatory fields

- **Audit Logs**: 100% of critical actions (as defined in Principle III) MUST be recorded

- **Audit Log Retention**: Minimum 7 years; logs MUST be immutable after creation

- **Reports Accuracy**: CSV/PDF exports MUST match backend data; tolerance ±0.01 for currency rounding only

- **API Response**: Tiered by operation type (see [Performance Standards](#performance-standards))

- **Data Integrity**: Cross-module reconciliation MUST complete within 5 minutes of transaction commit

- **Interface Clarity**: Forms MUST complete primary action in ≤5 clicks; error messages MUST specify field name + expected format

---

## Security Standards

### Authentication
| Control | Requirement |
|---------|-------------|
| Failed login lockout | 5 attempts → 15-minute lockout |
| Password minimum | 8 chars, 1 uppercase, 1 lowercase, 1 number, 1 special char |
| Password expiry | 90 days (configurable) |
| MFA | Required for admin roles |

### Session Management
| Control | Requirement |
|---------|-------------|
| Inactivity timeout | 30 minutes |
| Absolute timeout | 8 hours |
| Concurrent sessions | Max 3 per user |
| Session invalidation | On password change, forced logout |

### API Security
| Control | Requirement |
|---------|-------------|
| Authentication | Valid JWT required for all endpoints |
| Token expiry | 8 hours (access), 7 days (refresh) |
| Rate limiting | 100 requests/minute per user |
| CORS | Whitelist only configured origins |

### Data Protection
| Control | Requirement |
|---------|-------------|
| Sensitive data masking | Show only last 4 digits: bank accounts, tax IDs |
| Encryption at rest | AES-256 for sensitive fields |
| Encryption in transit | TLS 1.2+ required |
| PII in logs | MUST be masked or excluded |

### Access Control
| Control | Requirement |
|---------|-------------|
| RBAC | Every endpoint MUST check permissions before execution |
| Principle of least privilege | Default deny; explicit grants only |
| Permission audit | Quarterly review of user permissions |

---

## Module-Specific Rules

### Product Module
- **SKU Format**: `[CATEGORY]-[6-DIGIT-ID]` (e.g., `ELEC-000123`)
- **Status Transitions**: `Draft → Active → Discontinued` (no skip allowed)
- **Price Changes**: MUST log effective date, old price, new price, approver
- **Required Fields**: Name, SKU, Category, Unit of Measure, Base Price

### Purchase Module
- **Line Items**: Purchase Orders MUST have ≥1 line item
- **Approval**: Required for amounts > configured threshold (see Workflow Gates)
- **Supplier Validation**: Lead time MUST be tracked; alert if delivery exceeds lead time by >20%
- **Three-Way Match**: PO + Goods Receipt + Invoice MUST match before payment approval

### Production Module
- **BOM Validation**: Component availability MUST be verified before production start
- **Tracking**: Planned vs actual quantities, wastage %, downtime
- **Quality Gate**: QC status required before marking production complete
- **Costing**: Actual cost calculated from consumed materials + labor + overhead

### Sales Module
- **Stock Validation**: Orders MUST validate availability; backorders flagged explicitly
- **Pricing**: Pull from active price list; manual overrides require approval + documented reason
- **Invoicing**: Invoice MUST generate within 24 hours of delivery confirmation
- **Credit Check**: Orders exceeding customer credit limit require approval

### Accounting Module
- **Double-Entry**: Every transaction MUST have balanced debits = credits
- **Period Control**: Posting to closed periods MUST be blocked
- **Journal Approval**: Entries > configured threshold require supervisor approval
- **Reconciliation**: Bank reconciliation MUST be completed monthly

### Reporting Module
- **Timeout**: 30 seconds (standard), 120 seconds (complex reports)
- **Metadata**: All financial reports MUST include generation timestamp + data cutoff time
- **Retry Policy**: Scheduled reports retry 3x on failure before alerting
- **Access Control**: Sensitive reports (P&L, payroll) restricted by role

---

## Performance Standards

### API Response Times (p95)

| Operation Type | Target | Max Acceptable | Example |
|----------------|--------|----------------|--------|
| Single record GET | 200ms | 500ms | Get product by ID |
| List queries (≤100 items) | 500ms | 2s | Product list page 1 |
| List queries (100-1000 items) | 2s | 5s | Full inventory export |
| Search queries | 1s | 3s | Product search |
| Complex reports | 5s | 30s | Monthly P&L |
| Batch operations (≤100 items) | 5s | 15s | Bulk price update |
| Batch operations (>100 items) | 30s | 120s | Mass inventory adjustment |

### Database Performance
| Metric | Threshold |
|--------|----------|
| Query execution | No query >5s without optimization review |
| Index usage | All WHERE clauses MUST use indexes |
| Connection pool | Max 100 connections; alert at 80% |

### Report Generation
| Report Type | Max Time | Max Rows |
|-------------|----------|----------|
| Standard list | 10s | 10,000 |
| Financial summary | 30s | N/A |
| Complex analytical | 120s | 100,000 |
| Data export (CSV) | 60s | 50,000 |

---

## Testing Requirements

### Coverage Standards
| Test Type | Minimum Coverage | Enforcement |
|-----------|------------------|-------------|
| Unit tests | 80% for business logic | CI gate |
| Integration tests | Required for all cross-module operations | CI gate |
| Contract tests | Required for all API endpoints | CI gate |

### Regression Testing
- All tests MUST run on every PR
- All tests MUST pass before merge (zero tolerance)
- Flaky tests MUST be fixed or quarantined within 48 hours

### Load Testing
- **Frequency**: Quarterly, or before major releases
- **Baseline**: 100 concurrent users without degradation
- **Stress test**: Identify breaking point; document in release notes

### Data Validation Testing
- Automated reconciliation tests run nightly
- Cross-module balance checks (Inventory ↔ Accounting ↔ Sales)
- Alert on any discrepancy >±0.01

### Security Testing
- Dependency vulnerability scan: Weekly (automated)
- OWASP Top 10 review: Before each major release
- Penetration testing: Annually

---

## Workflow Compliance Gates

### Document State Machine

All business documents (PO, SO, Invoice, Journal Entry) MUST follow this state machine:

```
Draft → Submitted → Approved → Posted → [Closed | Cancelled]
              ↓
           Rejected → Draft
```

| Transition | Gate | Requirements |
|------------|------|-------------|
| Draft → Submitted | Validation | All required fields populated, references valid |
| Submitted → Approved | Authorization | Approver has sufficient limit, not self-approval for amounts >$1000 |
| Approved → Posted | Audit | Creates immutable audit record, updates GL |
| Posted → Cancelled | Reversal | Creates reversal entry; original preserved |

**Posted documents**: CANNOT be edited or deleted. Corrections via reversal entries only.

### Approval Thresholds

| Document Type | Self-Approve | Manager Required | Director Required |
|---------------|--------------|------------------|-------------------|
| Purchase Order | ≤$1,000 | $1,001 - $10,000 | >$10,000 |
| Sales Discount | ≤5% | 6% - 15% | >15% |
| Journal Entry | ≤$5,000 | $5,001 - $50,000 | >$50,000 |
| Inventory Adjustment | ≤10 units | 11 - 100 units | >100 units |
| Credit Limit Override | N/A | ≤$10,000 | >$10,000 |

*Thresholds are configurable per organization but MUST have defined values.*

### Code Review Gates

| Change Type | Min Approvals | Additional Requirements |
|-------------|---------------|------------------------|
| Standard PR | 1 | All CI checks pass |
| Financial calculations | 2 | Include test cases for edge values |
| Database migrations | 2 | Rollback script required |
| Security (auth, permissions) | 2 | Security review label required |
| API contract changes | 2 | Deprecation notice if breaking |

---

## Constraints

| Constraint | Limit | Enforcement |
|------------|-------|-------------|
| Records per table | 1,000,000 max | Database partitioning strategy required at 500K |
| Export rows (UI) | 10,000 max | Application validation |
| Export rows (API/scheduled) | 100,000 max | Pagination required |
| Attachment size | 10MB per file | Upload validation |
| Batch operation size | 1,000 items max | API validation |
| Report timeout | 120 seconds | Server-side timeout |
| Concurrent report jobs | 5 per user | Queue management |

---

## Success Criteria

- **SC-001 Accuracy**: Automated reconciliation tests pass with 0 discrepancies (tolerance ±0.01 for currency)
- **SC-002 Audit Completeness**: 100% of critical actions have corresponding audit log entries (verified by automated audit)
- **SC-003 Performance**: p95 response times within "Max Acceptable" thresholds; error rate <1%
- **SC-004 Usability**: Form submission error rate <5%; all workflows documented in help system
- **SC-005 Data Integrity**: Nightly cross-module reconciliation passes with 0 discrepancies
- **SC-006 Security**: Zero critical/high vulnerabilities in production; all security tests pass
- **SC-007 Test Coverage**: ≥80% unit test coverage; all integration tests pass

---

## Enforcement Rules

1. Constitution MUST be committed to Git before any module or feature development begins
2. All modules, specs, plans, tasks, and implementations MUST respect Constitution standards
3. No feature can bypass data validation, audit logs, or security constraints
4. Changes to Constitution MUST be reviewed and approved before implementation
5. All PRs/reviews MUST verify compliance with Constitution principles
6. Constitution violations discovered in code review MUST block merge until resolved
7. Performance degradation beyond thresholds MUST be treated as a bug (P1 priority)

---

## Governance

### Amendment Procedure

1. Proposed changes MUST be documented with rationale and impact analysis
2. Changes MUST be reviewed by project stakeholders (minimum 2 approvers)
3. Approved changes increment the version according to semantic versioning
4. All dependent artifacts MUST be updated to reflect changes
5. Teams MUST be notified of changes within 24 hours of ratification

### Versioning Policy

- **MAJOR**: Backward incompatible governance/principle removals or redefinitions
- **MINOR**: New principle/section added or materially expanded guidance
- **PATCH**: Clarifications, wording, typo fixes, non-semantic refinements

### Compliance Review

- Every feature specification MUST reference applicable Constitution principles
- Implementation plans MUST include a Constitution Check gate
- Code reviews MUST verify adherence to Constitution constraints
- Quarterly compliance audit: Review random sample of 10 PRs for Constitution adherence

---

*This Constitution provides the foundation for all work in the IHS ERP project. Every specification, plan, task, and implementation must adhere to these rules to maintain quality, consistency, and reliability across the system.*

**Version**: 1.1.0 | **Ratified**: 2025-12-10 | **Last Amended**: 2025-12-10
