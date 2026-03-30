# Roam Batch Actions Reference

`roam_process_batch_actions` is the primary write tool (canonical since v1.36.0). Accepts an array of action objects. All actions execute sequentially — build parents before children.

## create-block

```json
{
  "action": "create-block",
  "location": {
    "page": "<page-uid>",
    "order": "last"
  },
  "block": {
    "string": "block content",
    "uid": "optional-uid",
    "open": true,
    "heading": 0
  }
}
```

- `location.page` OR `location.block` — target a page or a parent block
- `order` — integer index, `"last"`, or `"first"`
- `uid` — 9-char alphanumeric if set manually; omit to auto-generate
- `heading` — `0` = none, `1` = h1, `2` = h2, `3` = h3

## update-block

```json
{
  "action": "update-block",
  "block": {
    "uid": "<block-uid>",
    "string": "new content",
    "open": true,
    "heading": 0
  }
}
```

## move-block

```json
{
  "action": "move-block",
  "location": {
    "page": "<page-uid>",
    "order": "last"
  },
  "block": {
    "uid": "<block-uid>"
  }
}
```

## delete-block

```json
{
  "action": "delete-block",
  "block": {
    "uid": "<block-uid>"
  }
}
```

## Notes

- You can mix action types in a single batch call.
- `order: "last"` appends; `order: 0` prepends.
- Parent blocks must be created before children in the same batch.
- `uid` in create-block must be exactly 9 alphanumeric characters if provided.
