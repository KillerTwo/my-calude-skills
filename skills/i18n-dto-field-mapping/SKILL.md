---
name: i18n-dto-field-mapping
description: Use when adding internationalized display fields to Java controller or service query results based on a source dictionary field and Def.xxx mapping, especially when the DTO may be missing the xxxText field or the source field type may not match the Def.xxx value type.
---

# I18n DTO Field Mapping

## Overview

Use this skill when a Java query result needs an internationalized display field derived from a dictionary field.

Preferred input is one natural-language description that includes:
- A controller interface or service method
- One or more field mappings such as `a->Def.A,b->Def.B,c->Def.C`

Typical inputs include:
- `对service1#listPage返回DTO进行国际化转换，字段映射关系如下：a->Def.A,b->Def.B,c->Def.C`
- `对controllerX#detail返回DTO进行国际化转换，字段映射关系如下：status->Def.Status`

The goal is to make the returned DTO expose both:
- The original field such as `a`
- The i18n key field such as `aText`
- The original field such as `b`
- The i18n key field such as `bText`

## When to Use

Use when:
- The user asks to add internationalized conversion for a query result DTO
- The user provides the request as one sentence with one or more field mappings
- The mapping rule is given as `Codes.getI18nKey(Def.xxx.class, value)`
- The source is a controller method or service method
- The result is a list query or a single-object query
- The DTO may be missing the `xxxText` field
- The source dictionary field type may not match the `Def.xxx` value type

Do not use when:
- The task is only about request parameters or persistence-layer enum definitions
- The user wants translated text values instead of i18n keys
- The method does not return a DTO that can be enriched before response

## Required Inputs

Before changing code, confirm these inputs from the user request or local code:
- Target controller interface or service method
- At least one field mapping in the form `a->Def.xxx`

Preferred parsing rule:
- Extract the target method from the natural-language description
- Extract every mapping pair from the mapping segment
- Treat each pair as one independent conversion task inside the same DTO flow

Example:
- Input: `对service1#listPage返回DTO进行国际化转换，字段映射关系如下：a->Def.A,b->Def.B,c->Def.C`
- Parsed method: `service1#listPage`
- Parsed mappings:
  - `a -> Def.A`
  - `b -> Def.B`
  - `c -> Def.C`

If the target method or the mapping list is missing, stop and ask for the missing input.

## Workflow

### 1. Locate the returned DTO

Identify the method return type from the target controller or service.

Handle common cases:
- `List<SomeDTO>`
- `PageResult<SomeDTO>`
- `IPage<SomeDTO>`
- `SomeDTO`
- Wrapper response types whose payload is one DTO or a DTO list

If the controller delegates to a service, trace the actual DTO returned to the response layer.

### 2. Process every mapping pair

Never stop after handling only the first mapping.

For each mapping pair such as:
- `a -> Def.A`
- `b -> Def.B`
- `c -> Def.C`

run the full workflow below for each source field in the same DTO.

### 3. Check the source field type first

Find the DTO field named `a`.

Then inspect the expected value type used by `Def.xxx`. The source field type and the `Def.xxx` value type must match. If they do not match, `Codes.getI18nKey(...)` may fail or return the wrong result.

Required rule:
- If the DTO source field type does not match the `Def.xxx` value type, change the DTO source field type first so they are consistent.

Examples:
- `Def.xxx` expects `Integer`, but DTO field `a` is `String`: change `a` to `Integer`
- `Def.xxx` expects `Long`, but DTO field `a` is `Integer`: change `a` to `Long`

After changing the field type, also check:
- Getter and setter signatures
- Builder or constructor parameters
- Mapping code that sets this field
- Serialization or conversion code affected by the type change

Apply the same rule independently to every mapped field.

### 4. Ensure the DTO contains `aText`

Compute the i18n field name by concatenation:
- Source field `a`
- Suffix `Text`

Example:
- `isActive` -> `isActiveText`

If the DTO already has that field, reuse it.

If not, add:

```java
@Schema(description = "是否启用国际化Key")
@I18nField
private String isActiveText;
```

Apply the same pattern for any other field name:
- Keep the type as `String`
- Keep `@I18nField`
- Use a description consistent with the business meaning of the field

Also verify the DTO has the corresponding getter and setter if the project style requires them explicitly.

Apply the same rule to every mapped field:
- `a -> aText`
- `b -> bText`
- `c -> cText`

### 5. Add i18n assignment logic

Use the source field value to populate `aText` with `Codes.getI18nKey(Def.xxx.class, value)`.

#### List query

If the method returns a query list, iterate over the list and assign the i18n key on each item:

```java
list.forEach(v -> {
    v.setAText(Codes.getI18nKey(Def.A.class, v.getA()));
    v.setBText(Codes.getI18nKey(Def.B.class, v.getB()));
    v.setCText(Codes.getI18nKey(Def.C.class, v.getC()));
});
```

Rules:
- Use the actual list variable name from the method
- Use the DTO getter for the source field
- Use the DTO setter for the `aText` field
- When there are multiple mappings, set all related `xxxText` fields in the same result iteration
- If the method already contains other field conversion logic, keep the existing conversion code and append only the newly requested field conversions

#### Single-object query

If the method returns one DTO, assign directly:

```java
dto.setAText(Codes.getI18nKey(Def.A.class, dto.getA()));
dto.setBText(Codes.getI18nKey(Def.B.class, dto.getB()));
dto.setCText(Codes.getI18nKey(Def.C.class, dto.getC()));
```

Rules:
- Guard against `null` when the surrounding method may return `null`
- Keep the assignment close to the place where the DTO is assembled or returned
- When there are multiple mappings, set every requested `xxxText` field before return
- If the method already contains other field conversion logic, keep the existing conversion code and append only the newly requested field conversions

### 6. Final verification

Before finishing, verify:
- Every mapping pair from the input was processed
- The DTO contains every source field such as `a`, `b`, `c`
- Every DTO source field type matches its `Def.xxx` value type
- The DTO contains every `xxxText` field such as `aText`, `bText`, `cText`
- Every `xxxText` field is `String`
- Every `xxxText` field carries `@I18nField`
- Every `Codes.getI18nKey(Def.xxx.class, value)` uses the correct getter value
- List queries use iteration
- Single-object queries use direct assignment
- Existing unrelated or previously implemented field conversion code was not deleted
- Imports compile cleanly after the change

## Output Standard

When applying this pattern, make changes in this order:
1. Parse the target method and all field mappings
2. For each mapping, adjust source field type if needed
3. For each mapping, add `xxxText` field if needed
4. Add all requested i18n assignments in the query result flow
5. Re-check DTO accessors and compile references affected by all type changes

## Batch Mapping Rule

If the input contains multiple mappings, treat them as one batch task on the same returned DTO.

Example:
- Input: `对service1#listPage返回DTO进行国际化转换，字段映射关系如下：a->Def.A,b->Def.B,c->Def.C`

Expected behavior:
- Find the DTO returned by `service1.listPage`
- Process `a -> Def.A`
- Process `b -> Def.B`
- Process `c -> Def.C`
- Do not stop after completing only one field
- Produce one coherent DTO and one coherent assignment block for the method
- Preserve any existing conversion statements for other fields and only add the requested new ones

## Incremental Update Rule

This skill performs additive changes.

If the target method already contains conversion code for other fields:
- Do not delete existing conversion statements
- Do not replace existing unrelated `xxxText` assignments
- Do not shrink an existing conversion block just to fit the new fields
- Append only the newly requested field conversions in the existing result-processing flow

If the DTO already contains other i18n fields:
- Keep the existing fields
- Add only the missing requested `xxxText` fields

## Common Mistakes

- Parsing only the method and ignoring some mapping pairs
- Handling only `a` and forgetting `b` or `c`
- Deleting existing conversion code while adding the new requested mappings
- Adding `aText` but forgetting to populate it
- Populating `aText` with the wrong `Def.xxx`
- Modifying only the setter logic without checking the DTO field type
- Leaving DTO field `a` as `String` when `Def.xxx` expects `Integer` or `Long`
- Applying list iteration logic to a single-object query
- Applying direct assignment to a collection result
- Updating the DTO field declaration but forgetting related getter or setter signatures

## Default Pattern

Use this decision rule:
- Query returns multiple DTOs: iterate and set `aText` for each item
- Query returns one DTO: assign `aText` directly

Always treat source field type compatibility with `Def.xxx` as a prerequisite, not as a cleanup step after the i18n code is added.
If the input contains multiple mappings, apply the same decision rule to every mapping pair in one pass.
