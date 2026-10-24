# Glossary

Status: Supporting reference
Normative source: [../core/00-rules.md](../core/00-rules.md)

Terms are alphabetical. When a definition conflicts with the core rules, the core rules prevail.

## A

**Absence.** A normal query outcome with no matching record. Distinct from failure. `find` represents it as a value; `get` treats it as an error; `tryGet` returns an explicit result.

**Adapted profile.** A profile with one or more configurable choices different from the default, all core rules satisfied, and every choice recorded. Label: adapted, subject to validation.

**Aggregate.** The domain object owned by a repository contract. In `IUserRepository`, the aggregate is `User`.

**Aggregate noun.** The noun in the enclosing type name that identifies the aggregate. Members do not repeat it.

**Aggregate-scoped interface.** An interface, trait, protocol, or abstract class whose name establishes the aggregate it operates on.

**All-based profile.** An adapted profile that replaces the `One`/`Many` vocabulary for read, update, and removal scope with an implicit single form and `All` for plural scope. `insertOne`/`insertMany` are retained.

**Audit.** The read-only phase of R10 that inventories contracts, call sites, tests, constraints, and violations. It produces a candidate level, not a supported label.

## B

**Bare single update.** A single update with no `By` segment, permitted only when a required parameter carries the aggregate root's unique id. `updateOne(user)` and `updateOne(id, patch)` are valid; `updateOne(patch)` and `updateOne()` are not. `updateOneById` is preferred.

**Breaking change.** A change that can break source, binary, public API, public behavior, serialized data, reflection, configuration, documentation, or tests. Classified as none, internal, or public.

**`By` segment.** The part of a method name beginning with `By` that encodes an explicit condition, such as `ById`, `ByStatusAndTenantId`, or `ByIdIn`.

## C

**Cardinality.** The number of targets: at most one (`One`), zero or more (`Many`), or all in an approved All-based profile. Must be distinguishable from the name and return type.

**Characterization test.** A test that pins existing behavior before a refactor.

**Compliance label.** Strict, Adapted, Partial, Non-compliant, or Audit-only. A conclusion supported by evidence; a core waiver prevents Strict or Adapted.

**Condition.** A predicate selecting records. Explicit conditions use `By`; a single update's intrinsic identity is implicit.

**Core rule.** A rule that must not be waived while claiming compliance: R1–R7, and R10. R6 grammar is mandatory; surface conventions are configurable.

## D

**Declared visibility scope.** The records a repository may read or affect, such as a tenant or policy boundary. `findMany()` means all records in this scope.

**Decision register.** The durable record of adaptations and waivers. Fields: ID, Rule, Default, Decision, Rationale, Scope, Consequence, Approver, Date, Review.

**Default profile.** `insert`/`find`/`update`/`remove`, `One`/`Many`, no `All`, `find`-first reads, `updateOneById` preferred.

**Drift.** Divergence between adopted rules and the code, register, or emitted block.

## E

**Emission.** The controlled write of the rule set into an instruction file. Owns one managed block.

**Enforcement.** A repeatable check that prevents regression: CI scan, analyzer, architecture test, review checklist, or contract snapshot.

**Explicit condition.** A non-identity predicate such as status, email, tenant, or range. Must use `By`.

## F

**Find.** The default read family. `findOne*` returns a record or absence value; `findMany*` returns a non-null collection. Absence is not an error.

**Find-first.** The priority `find > tryGet > get`.

## G

**Get.** A read family that treats absence as an error. `getOne*` returns the record or a typed error; never an absence value. Service or boundary contract by default; a repository opt-in requires an R8 entry.

## I

**Identity-bearing update.** A single update whose required parameter carries the aggregate root's unique id. The id is the implicit condition; the repository rejects a missing or invalid id.

**Infrastructure parameter.** A parameter supporting execution rather than selection: cancellation token, transaction, trace context.

## L

**Lifecycle operation.** A documented operation distinct from CRUD, such as `archiveOneById`, `restoreOneById`, `publishOneById`, or `purgeOneById`.

## M

**Managed block.** The region delimited by the CRUD Scope Standardizer markers. The skill owns only this region.

**Many.** The scope token for zero or more targets. Preferred over `All`. `findMany()` returns the declared full scope.

**Multiple-match violation.** A `findOne` or `getOne` condition matching more than one record. A contract violation, not a reason to return the first row.

## N

**Not-found error.** A typed error signaling absence where the contract requires the record. Identifies the aggregate and lookup key; no secrets or raw database errors.

## O

**One.** The scope token for at most one target. Requires a uniqueness-guaranteeing condition.

## P

**Partial profile.** A profile with a core rule waived or violated. Label: partially aligned; not compliant.

**Presentation parameter.** Ordering, pagination, projection, or transport parameter. Does not count as a condition.

**Projection / read model.** A type representing a subset or combination of data. Cross-aggregate reads should use a dedicated repository or read model.

## R

**Repository contract.** An interface, trait, protocol, or abstract class exposing persistence operations for one aggregate.

**Rule catalog.** The normative source of truth: [../core/00-rules.md](../core/00-rules.md).

## S

**Scalar helper.** Optional `count` or `exists` operation. Does not replace a CRUD verb and does not use `One`/`Many`/`All` tokens.

**Semantic parameter.** A parameter changing which records are selected or changed: id, status, patch, filter, specification.

**Singleton aggregate.** An aggregate whose contract guarantees exactly one instance. May expose `findOne()` without a condition when documented.

**Strict profile.** Default profile, all core rules satisfied, no adaptations or waivers. Label requires validation evidence.

## T

**Transport suffix.** A framework-mandated suffix such as `Async`, placed after the semantic name.

**TryGet.** A read family returning an explicit success-or-absence result without throwing for absence. Ranks below `find` and above `get`.

## U

**Uniqueness-guaranteeing condition.** A condition matching at most one record, such as an id or complete unique key.

**`updateOneById`.** The preferred explicit single-update form. A bare single update is permitted only with a required identity parameter.

## V

**Validation report.** The evidence artifact recording scope, profile, checks, findings, waivers, label, and sign-off.

## W

**Waiver.** An explicit, recorded decision to accept a core-rule deviation. Forces the partial profile and must state the lost guarantee.
