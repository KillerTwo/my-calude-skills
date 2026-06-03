---
name: dynamic-query-condition-mapping
description: Use when adding dynamic query conditions to Java controller or service query methods based on entity-field match rules, especially when the request is expressed as Entity#field->matchType and the changes must be applied to query parameter DTOs and MyBatis dynamic SQL without deleting existing conditions.
---

# Dynamic Query Condition Mapping

## Overview

Use this skill when a Java query interface needs new dynamic query conditions added to its query parameter DTO and MyBatis dynamic SQL.

Preferred input is one natural-language description that includes:
- A target controller method or service method
- One or more field mappings in the form `Entity#field->matchType`

Typical inputs include:
- `给接口MaterialDeviceController#pageDevices方法添加如下动态查询条件字段：MaterialDevice#mnemonicCode->模糊匹配、MaterialDevice#deviceType->完全匹配、MaterialDevice#validationExpiryDate->范围匹配[startDate,endDate)`
- `给MaterialDeviceService#pageDevices方法添加动态查询条件：MaterialDevice#deviceType->完全匹配`

The goal is to:
- Find the query parameter DTO used by the target method
- Add the requested fields to that DTO when missing
- Find the MyBatis dynamic query SQL
- Convert entity and field names into table and column conditions
- Append only the requested new query conditions

## When to Use

Use when:
- The user asks to add dynamic query conditions to an existing Java query method
- The request is expressed with one or more mappings such as `Entity#field->模糊匹配`
- The target is a controller method or service method
- The query uses a DTO or VO as request parameters
- The filtering logic is implemented in MyBatis XML or equivalent dynamic SQL
- Existing query conditions must be preserved while adding new ones

Do not use when:
- The task is about response DTO fields instead of query parameters
- The task is about static SQL with no dynamic condition flow to extend
- The request does not identify a method or any field mapping

## Required Inputs

Before changing code, confirm these inputs from the user request or local code:
- One target method in the form `Controller#method` or `Service#method`
- At least one mapping in the form `Entity#field->matchType`

Preferred parsing rule:
- Extract the target method from the natural-language description
- Extract every mapping pair from the mapping segment
- Treat each pair as one independent query-condition task in the same method flow

Example:
- Input: `给接口MaterialDeviceController#pageDevices方法添加如下动态查询条件字段：MaterialDevice#mnemonicCode->模糊匹配、MaterialDevice#deviceType->完全匹配、MaterialDevice#validationExpiryDate->范围匹配[startDate,endDate)`
- Parsed target method: `MaterialDeviceController#pageDevices`
- Parsed mappings:
  - `MaterialDevice#mnemonicCode -> 模糊匹配`
  - `MaterialDevice#deviceType -> 完全匹配`
  - `MaterialDevice#validationExpiryDate -> 范围匹配[startDate,endDate)`

If the method or mapping list is missing, stop and ask for the missing input.

## Workflow

### 1. Trace the target method and parameter DTO

If the input gives a controller method:
- Find the controller method
- Identify the request parameter DTO used by that method
- Follow the call into the service method if needed

If the input gives a service method:
- Find the service method
- Identify the request parameter DTO used by that method
- Trace back to the controller only when needed to confirm the entry DTO

Common cases:
- Controller method directly accepts `QueryDTO`
- Controller method accepts request object and forwards it to service
- Service method accepts `QueryDTO`, `PageDTO`, or a subclass

The skill must be able to continue when the user provides only one side.

### 2. Process every mapping pair

Never stop after handling only the first mapping.

For each mapping pair such as:
- `MaterialDevice#mnemonicCode -> 模糊匹配`
- `MaterialDevice#deviceType -> 完全匹配`
- `MaterialDevice#validationExpiryDate -> 范围匹配[startDate,endDate)`

run the full workflow below in the same parameter DTO and SQL flow.

### 3. Add missing DTO query fields

Find the DTO that carries the query parameters.

For each requested mapping:
- Check whether the DTO already contains the required query field
- If not, add it

#### Exact field naming rule

If the mapping is:
- `Entity#field -> 模糊匹配`
- `Entity#field -> 完全匹配`

then add the DTO field using the original Java field name:

```java
private String mnemonicCode;
private Integer deviceType;
```

If the mapping is:
- `Entity#field -> 范围匹配[startField,endField)`

then add range fields using the names explicitly provided in the request:

```java
private LocalDate startDate;
private LocalDate endDate;
```

Rules:
- Reuse existing fields when already present
- Keep the DTO field type consistent with the entity field type and existing project conventions
- For range queries, check whether both boundary fields already exist before adding anything
- If one range boundary exists and the other does not, add only the missing one

Also verify related code after DTO changes:
- Getter and setter methods when explicit accessors are required
- Lombok or inheritance conventions used by the project
- Validation or schema annotations if the surrounding DTO style requires them

### 4. Resolve entity to table and field to column

Use the entity name from `Entity#field` to locate the database table used by the query.

Required rule:
- Find the Java entity class
- Confirm the corresponding database table name
- Use the actual SQL column name for the requested field

Example:
- `MaterialDevice` -> `material_device`
- `mnemonicCode` -> `mnemonic_code`
- `deviceType` -> `device_type`
- `validationExpiryDate` -> `validation_expiry_date`

Do not guess blindly if the mapper or entity annotations show a different table or column mapping.

### 5. Find the MyBatis dynamic SQL

Locate the mapper XML or equivalent SQL fragment used by the target method.

Check:
- The mapper method called by the service
- The XML `<select>` or included SQL fragment
- Existing `<if>`, `<where>`, `<trim>`, or wrapper-based conditions

Add the new conditions in the same dynamic query flow used by the method.

### 6. Append query conditions by match type

#### Fuzzy match

For `Entity#field->模糊匹配`, add a `like` condition using the DTO field.

Typical pattern:

```xml
<if test="mnemonicCode != null and mnemonicCode != ''">
    AND mnemonic_code LIKE CONCAT('%', #{mnemonicCode}, '%')
</if>
```

#### Exact match

For `Entity#field->完全匹配`, add an equality condition using the DTO field.

Typical pattern:

```xml
<if test="deviceType != null">
    AND device_type = #{deviceType}
</if>
```

If the field type is `String`, the test should also guard against empty string when that matches project style:

```xml
<if test="deviceType != null and deviceType != ''">
    AND device_type = #{deviceType}
</if>
```

#### Range match

For `Entity#field->范围匹配[startField,endField)`, add range conditions using the explicit boundary field names from the request.

Preferred half-open pattern:

```xml
<if test="startDate != null">
    AND validation_expiry_date <![CDATA[>=]]> #{startDate}
</if>
<if test="endDate != null">
    AND validation_expiry_date <![CDATA[<]]> #{endDate}
</if>
```

If the local project already uses another equivalent range style, follow the existing project pattern, but preserve the requested semantic `[startDate,endDate)`.

### 7. Preserve existing query conditions

This skill performs additive changes.

If the target SQL already contains other query conditions:
- Do not delete existing `<if>` blocks
- Do not replace unrelated conditions
- Do not rewrite the whole dynamic SQL block unless required by local style
- Append only the newly requested conditions in the existing query flow

If the parameter DTO already contains other query fields:
- Keep the existing fields
- Add only the missing requested fields

### 8. Final verification

Before finishing, verify:
- Every mapping pair from the input was processed
- The correct query parameter DTO was updated
- Every requested DTO field exists
- Every requested range boundary field exists when range matching is requested
- Entity-to-table mapping is correct
- Java field to SQL column mapping is correct
- Every requested SQL condition was added with the correct match semantics
- Existing unrelated query fields were not deleted
- Existing unrelated SQL conditions were not deleted
- Controller and service method flow still uses the updated DTO correctly
- Imports compile cleanly after the DTO changes

## Match Type Rules

Supported match types:
- `模糊匹配`
- `完全匹配`
- `范围匹配[startField,endField)`

Interpret them as:
- `模糊匹配` -> `LIKE`
- `完全匹配` -> `=`
- `范围匹配[startField,endField)` -> `>= startField` and `< endField`

If the user gives a range match, always use the boundary field names from the request instead of inventing new names.

## Output Standard

When applying this pattern, make changes in this order:
1. Parse the target method and all mapping pairs
2. Trace the actual query parameter DTO
3. Add missing DTO query fields
4. Trace the mapper method and dynamic SQL
5. Resolve entity and field mappings to actual table and column names
6. Append all requested dynamic query conditions
7. Re-check method flow, DTO references, and SQL consistency

## Batch Mapping Rule

If the input contains multiple mappings, treat them as one batch task on the same query flow.

Example:
- Input: `给接口MaterialDeviceController#pageDevices方法添加如下动态查询条件字段：MaterialDevice#mnemonicCode->模糊匹配、MaterialDevice#deviceType->完全匹配、MaterialDevice#validationExpiryDate->范围匹配[startDate,endDate)`

Expected behavior:
- Find `MaterialDeviceController#pageDevices`
- Trace the query parameter DTO
- Add `mnemonicCode` if missing
- Add `deviceType` if missing
- Add `startDate` and `endDate` if missing
- Find the dynamic SQL for the same query path
- Append `mnemonic_code LIKE`
- Append `device_type =`
- Append `validation_expiry_date >= startDate` and `< endDate`
- Preserve any existing query fields and SQL conditions

## Common Mistakes

- Parsing only the first mapping and ignoring the others
- Adding query fields to the response DTO instead of the request DTO
- Guessing the table name without checking the entity mapping
- Using Java field names directly in SQL without converting to the real column name
- Implementing range match as closed interval when the request requires `[start,end)`
- Deleting existing query conditions while adding new ones
- Adding both range fields even when one already exists
- Updating the DTO fields but forgetting the XML condition blocks

## Default Pattern

Use this decision rule:
- `模糊匹配`: add one DTO field and one `LIKE` condition
- `完全匹配`: add one DTO field and one `=` condition
- `范围匹配[startField,endField)`: add two boundary DTO fields and two range conditions

If the input contains multiple mappings, apply the same decision rule to every mapping pair in one pass.
