# Proposed schema extensions

The Azure Pipelines YAML format includes 3 non-standard features of YAML.
With appropriate schema support, they should be possible to support in this language server.
Fortunately, JSON Schema allows extension.

## Initial key in a mapping

YAML mappings are unordered.
But, for security and performance reasons, some Azure Pipelines objects require that a particular key appear first.
This tells the parser what remaining keywords are acceptable.

```yaml
# In standard YAML, these two constructs are equivalent:

- keyA: valueA
  keyB: valueB

- keyB: valueB
  keyA: valueA

# In Azure Pipelines YAML, these two constructs are not equivalent:
- task: Foo@1
  displayName: Foo Task

- displayName: Foo Task # not valid: "task" must appear first
  task: Foo@1

# NB: `task` has peers like `script`, `bash`, and others that must be
#     recognized in a oneOf context
```

Normal JSON Schema to validate the above:

```json
{
    "properties": {
        "task": {
            "type": "string",
            "pattern": "^[A-Za-z_.]+@[0-9]+$"
        },
        "displayName": {
            "type": "string"
        }
    },
    "additionalProperties": false,
    ...
}
```

In order to support the notion that one key has to appear first, both for validation and Intellisense suggestions, we introduce a schema extension `firstProperty`.

```json
{
    "properties": {
        "task": {
            "type": "string",
            "pattern": "^[A-Za-z_.]+@[0-9]+$"
        },
        "displayName": {
            "type": "string"
        }
    },
    "additionalProperties": false,
    "firstProperty": "task"  // new construct
    ...
}
```

This schema only validates if the `firstProperty` keyword appears first.
For Intellisense suggestions, if the `firstProperty` keyword hasn't already been recognized at this level, only it will be suggested.
This latter behavior aggregates: if an `anyOf` construct includes several schema branches with `firstProperty` indicators, the set of those will be offered for Intellisense.