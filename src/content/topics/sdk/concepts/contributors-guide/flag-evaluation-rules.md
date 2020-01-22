---
title: "Flag evaluation rules in server-side SDKs"
excerpt: ""
---
## Overview
This topic explains the algorithm used to evaluate a feature flag. This document is intended to be used by developers contributing or building server-side SDKs for LaunchDarkly. For client-side SDKs, flag evaluation is handled by LaunchDarkly-- there is no need to implement this algorithm. To learn more about the different types of SDK, read [Client-side and server-side SDKs](./client-side-and-server-side).

LaunchDarkly's server-side SDKs evaluate flags for specific users. The arguments needed to evaluate a flag are the _flag key_, _user_, and the _default variation value_.

Evaluating a flag for a specific user involves the following steps:


1. [Preliminary checks](#preliminary-checks): for example, verifying that the flag is on and the supplied user has a key
2. [Prerequisite checks](#prerequisite-checks): determine whether the flag's prerequisites are met
3. [Individual targeting checks](#individual-targeting-checks): determine whether the user matches an individual user targeting rule
4. [Targeting rule checks](#targeting-rule-checks): determine whether the user matches other targeting rules
5. [Fallthrough](#fallthrough): return the default variation value

The result of the evaluation is:


1. The evaluated variation value
2. The evaluation reason for returning that variation

Note that variation values are often referenced by their *index*. For example, if a flag has two variations - `true` and `false` - `true` will be variation index 0, and `false` will be variation index 1.

As a side effect of the evaluation, the SDKs send events back to LaunchDarkly to report that the evaluation has occurred. The data sent back for each evaluation (including prerequisites) follows the schema described in the [schema reference guide for data export](./data-export-schema-reference).

## <a name="preliminary-checks"></a>Preliminary checks
Five preliminary checks occur before the main flag evaluation algorithm executes:

- If the SDK is offline, return the provided default value with the `ERROR` evaluation reason and the corresponding error code
- Fetch the flag definition from the flag store; if not found, return the provided default value with the `ERROR` evaluation reason and the corresponding error code
- If the user doesn't have a `key` attribute, return the provided default value with the `ERROR` evaluation reason and the corresponding error code
- If the user key is an empty string, log a warning and proceed
- If the flag is turned off, return the flag's off variation value with the `OFF` evaluation reason

If all the above checks pass, the evaluation algorithm proceeds with prerequisite checks.

## <a name="prerequisite-checks"></a>Prerequisite checks
A flag may have prerequisites. A prerequisite is defined by a tuple of prerequisite flag key and variation index. A prerequisite is satisfied if, for a given user, the prerequisite flag evaluates to the specified variation index.

In the prerequisite check step, the SDK should iterate through each flag's prerequisites and verify that each prerequisite is satisfied. If a prerequisite is not satisfied, the SDK should return the flag's off variation value, with the `PREREQUISITE_FAILED` evaluation reason and the prerequisite flag key that failed. Prerequisite evaluation is short-circuited; in other words, the SDK should return after the first failure.

Below are the steps for prerequisite evaluations:

- Retrieve the prerequisite flag; if not found, the prerequisite has failed
- If the prerequisite flag is off, the prerequisite has failed
- Evaluate the prerequisite flag for the given user (no default value)
- If the prerequisite flag evaluation result's variation index matches the variation index defined for the prerequisite, the prerequisite has been satisfied

Prerequisite flags themselves may also have their own prerequisites - the evaluations are recursive. Note that LaunchDarkly prevents recursive cycles during flag creation and modification.

Once all prerequisites have passed, or there were no prerequisites, the evaluation algorithm proceeds with individual targeting checks.

## <a name="individual-targeting-checks"></a>Individual targeting checks
Flags may have targets. A target is a list of user keys associated with a specific variation index. SDKs iterate through flags' targets and check whether the targets contain the given user's key.

If a target contains the user's key, return the associated variation with the `TARGET_MATCH` evaluation reason.

If there were no targets, or no targets matched the user, the evaluation algorithm proceeds with the targeting rule checks.

## <a name="targeting-rule-checks"></a>Targeting rule checks
Flags may have rules. Rules offer more complex ways to match arbitrary user attributes and segments, and to execute controlled rollouts based on those criteria. A rule is defined as a tuple of the ID, collection of clauses, and either the variation index or a rollout plan (described in more detail below). For a rule to match, all rules' clauses must be satisfied.

SDKs will iterate through flags' rules to find the first rule that matches the given user. If the matched rule has a variation index, return the corresponding variation, with the rule's ID and the `RULE_MATCH` evaluation reason. If, on the other hand, the rule has a rollout, follow the rollout logic to arrive at the variation and also return the corresponding variation, along with the rule ID and the `RULE_MATCH` evaluation reason.

If there were no rules, or no rules matched the user, the evaluation algorithm proceeds with the fallthrough.

## <a name="clauses"></a>Clauses
Rules have clauses, all of which must be satisfied for a rule to match the user. Clauses are defined as a tuple of:

- attribute: user attribute to consider for comparison
- operator: comparison to perform (see below)
- values: list of arbitrary values to compare to
- negate: boolean, whether to negate the outcome of clause's comparison (note that in the LaunchDarkly UI, negated clauses are represented as separate negated operators)

Evaluating a clause is a matter of taking the value of the given user attribute and performing the given operator comparison against the given values, and then applying whether the outcome should be negated. If this produces a positive result, the clause has been satisfied.

If the user attribute value is an array (or list), then the comparison is performed for each of those attribute values, against each of the clause values. In this case, a single user attribute value match to a single clause value is enough to satisfy the clause.

### <a name="operators"></a>Operators
[block:parameters]
{
  "data": {
    "h-0": "Operator",
    "h-1": "Argument Types",
    "h-2": "Condition",
    "0-0": "`in`",
    "0-1": "Any",
    "0-2": "The user attribute value exactly matches the clause value, including its type",
    "1-0": "`endsWith`",
    "1-1": "String",
    "1-2": "Either the clause value is an empty string, or the user attribute value ends with the clause value",
    "2-0": "`startsWith`",
    "2-1": "String",
    "2-2": "The user attribute value starts with the clause value",
    "3-0": "`matches`",
    "3-1": "String",
    "3-2": "The clause value, evaluated as a regex, matches the user attribute value",
    "4-0": "`contains`",
    "4-1": "String",
    "4-2": "The user value attribute value contains the clause value",
    "5-0": "`lessThan`",
    "5-1": "Number",
    "5-2": "The user attribute value is less than the clause value",
    "6-0": "`lessThanOrEqual`",
    "6-1": "Number",
    "6-2": "The user attribute value is less than or equal to the clause value",
    "7-0": "`greaterThan`",
    "7-1": "Number",
    "7-2": "The user attribute value is greater than the clause value",
    "8-0": "`greaterThanOrEqual`",
    "8-1": "Number",
    "8-2": "The user attribute value is greater than or equal to the clause value",
    "9-0": "`before`",
    "9-1": "Integer or string parsed as timestamp",
    "10-1": "Integer or string parsed as timestamp",
    "10-0": "`after`",
    "9-2": "The user attribute value less than the clause value",
    "10-2": "The user attribute value greater than the clause value",
    "11-1": "String parsed as semantic version",
    "12-1": "String parsed as semantic version",
    "13-1": "String parsed as semantic version",
    "11-0": "`semVerEqual`",
    "12-0": "`semVerLessThan`",
    "13-0": "`semVerGreaterThan`",
    "11-2": "The user attribute value and the clause value are semantically equivalent",
    "12-2": "The user attribute value precedes the clause value",
    "13-2": "The clause value precedes the user attribute value",
    "14-0": "`segmentMatch`",
    "14-1": "String",
    "14-2": "Segment match operation succeeds (see below)"
  },
  "cols": 3,
  "rows": 15
}
[/block]

### <a name="segments"></a>Segments

Clauses' `segmentMatch` operator allows for more fine-grained control of segments of users. The clause values are segment keys. To determine if this clause is satisfied, retrieve each of the segments identified by those keys and determine if the user is part of any of those segments.

Segments' properties include a list of included users, a list of excluded users, and a list of rules of their own, which themselves contain rule clauses with one exception: segment rule clauses may not contain `segmentMatch` operators.

To evaluate whether a user is in a segment:

- If the user's key is in the segment's `included` list, the result is `true`
- If the user's key is in the segment's `excluded` list, the result is `false`
- If neither, then iterate over the segment's rules to determine if any match
    - Iterate over the segment rule's clauses
        - Evaluate if the clause matches the user
        - If any clause doesn't match, the rule doesn't match and move on to the next rule
    - If the segment rule's weight is `null`, the rule matches and the result is `true`
    - Get the segment rule's `bucketBy` value; if not set, use `key`
    - Compute the bucket for the user using the segment rule's bucketing key and the segment's salt; otherwise, follow the same logic as for rollouts
    - If the user's bucket is less than the segment rule's weight divided by 100,000, the rule matches and the result is `true`; otherwise, the rule doesn't match and move on to the next rule
- If the user's key wasn't in the included and excluded lists, and none of the rules matched, the result is `false`

Finally, remember to return the potentially negated (depending on clause's `negate` property) result from above.

## <a name="rollouts"></a>Rollouts
Rules may be associated with rollouts. Rollouts provide flexibility to bucket users. They are defined as a tuple of a list of weighted variations and a name of the attribute by which to bucket the user. The weighted variations are defined as a tuple of variation index and a non-negative integer between 0 and 100,000 acting as that variation's weight.

To bucket the given user and calculate the final variation:

- Get the user's attribute name from the rollout (to bucket the user by), if not set use the user's key
- Get the user's attribute value
- Get the user's optional secondary identifier
- Compute the bucket for user
    - Concatenate the flag's key, the flag's salt, the user's attribute value and the secondary identifier (if any), joined by "." character
    - Take the first 
1. characters of the sha1 of the above
    - Convert the resulting base 
1. integer to a base 
1. integer
    - Divide the resulting base 
1. integer by `0xFFFFFFFFFFFFFFF` (`1152921504606846975`)
    - The result of this division is the user's bucket number
- Iterate over the rollout's weighted variations
    - Starting at 0, keep adding the weighted variation's weight divided by 100,000 to the sum
    - As soon as user's bucket is less than the above sum, return the weighted variation's variation index

## <a name="fallthrough"></a>Fallthrough
Fallthrough is the last step in flag evaluation. If the evaluation process is made here, it means that all prerequisite checks passed and none of the targets or rules matched the user.

Fallthrough is defined as a single variation of a rollout. This is similar to how rules are defined, except without iterating through any IDs or clauses. If fallthrough is a variation index, then return the corresponding variation value with the `FALLTHROUGH` evaluation reason. Otherwise, go through the same rollouts steps to arrive at the variation index, and again, return the corresponding value and the `FALLTHROUGH` evaluation reason.