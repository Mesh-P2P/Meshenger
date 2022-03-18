## Chats/Groups

The first message is a full config message, the Hash (of previous message) set to 0. There has to be at least one role with all permissions.

### Message distribution
TODO

## Collissions
If two messages are sent with the same index the following gets kept:
1. the message that has another linked to it via hash
2. the message with the lower timestamp
3. the message with the smaller SHA-256 hash

## Message Data
| Byte      | 0-16    | 16-17 | 17-n      |
|-----------|---------|-------|-----------|
| Content   | Chat ID | Type  | Type Data |
| Data type | UUID    | UInt8 | Type Data |

### Raw Message (Type 0)
| Byte      | 0-n       | n-m                           | m-k                        | k-(k+8)        | (k+8)-(k+12) | (k+12)-l     |
|-----------|-----------|-------------------------------|----------------------------|----------------|--------------|--------------|
| Content   | Sender ID | Signature (of following Data) | Hash (of previous message) | Unix timestamp | Index        | JSON Message |
| Data type | ID        | Sig                           | hash                       | UInt64         | UInt32       | JSON (utf8)  |

#### JSON Message
```javascript
{
  type: "type.subtype.subsubtype",
  body,
  reply: index, //(optional)
}
```

#### Standard Types
| Type             | body data                   |
|------------------|-----------------------------|
| text             | Markdown                    |
| file             | {hash, name, filetype}      |
| read             | Index of the message        |
| delete           | Index of the message        |
| config           | [options](#config_messages) |
| contact request  | nothing                     |
| contact response | accept? (bool)              |

#### Config Messages
only send changes
```javascript
{
  hash: "Hash algorithm",
  name: "name",
  members: [
    {
      id,
      pub, //(if Mesh IDs are not just the key)
      name: "name",
      role: "role"
    }
  ],
  roles: {
    "role": {
      invite,
      remove,
      changeRoles,
      changeHash
    }
  }
}
```

### File Available (Type 1)
| Byte      | 0-4                   |
|-----------|-----------------------|
| Content   | Index of file message |
| Data type | UInt32                |

### File Request (Type 2)
| Byte      | 0-4                   |
|-----------|-----------------------|
| Content   | Index of file message |
| Data type | UInt32                |

### File (Type 3)
| Byte      | 0-4                   | 4-n      |
|-----------|-----------------------|----------|
| Content   | Index of file message | File     |
| Data type | UInt32                | Raw data |

### Get Messages after Index (Type 4)
| Byte      | 0-4    |
|-----------|--------|
| Content   | Index  |
| Data type | UInt32 |

### Messages after Index (Type 5)
| Byte      | 0-1   | 1-n             | n-(n+1) | (n+1)-m           | ... |
|-----------|-------|-----------------|---------|-------------------|-----|
| Content   | Size  | Message (Index) | Size    | Message (Index+1) | ... |
| Data type | UInt8 | Type 0          | UInt8   | Type 0            | ... |

### Reject Message (Type 6)
| Byte      | 0-4              |
|-----------|------------------|
| Content   | Index of message |
| Data type | UInt32           |
