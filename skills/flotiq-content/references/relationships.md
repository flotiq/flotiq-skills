# Wiring up relationships in content

This is about populating relations that already exist in the schema. For designing relation fields themselves, see the `flotiq-schema` skill's [`relations-and-media.md`](../../flotiq-schema/references/relations-and-media.md).

## Resolve before you relate

* Never invent or guess a related object's ID. Look up the target object (by a known field like slug or name) or create it first if it genuinely doesn't exist yet.
* When creating a batch of objects with relations between them (e.g. posts and authors), create the "one" side of a one-to-many relation first, then create the "many" side referencing it.

## Avoid accidental duplication

* Before creating a new related object (a new `Author`, a new `Category`), check whether one matching it already exists. Creating "Jane Doe" as an author twice under slightly different objects fragments content and breaks the point of using a relation.

## Required relations

* If a relation is required by the schema, the object can't be meaningfully created without it — resolve the dependency first rather than leaving the object incomplete or using a placeholder.

## Verifying relations

After setting a relation, re-fetch the object and confirm the relation resolves to the intended target — an object with the right ID but wrong content type, or a stale reference, will still look "set" without being correct.
