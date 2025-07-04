# API Key System 🔑

## What is an API Key?

An API Key is simply a **long, random string**. No JWT, no signature - just pure randomness!

```go
// This simple:
func generateAPIKey() string {
    bytes := make([]byte, 32) // 32 bytes = 256 bits
    crypto/rand.Read(bytes)

    // Convert to hex
    key := hex.EncodeToString(bytes)

    // Add prefix for readability
    return "dpl_" + key
}

// Result: dpl_8f3d2a9b5c7e1f4a6d8b3c5e7f9a2d4e6b8c1d3f5a7b9c2e4f6a8d1b3c5e7f9
```

## How is it Stored? 🗄️

**NEVER in plaintext!** Always as a hash:

```go
// During creation:
apiKey := generateAPIKey()           // dpl_8f3d2a9b5c...
hash := sha256.Sum256([]byte(apiKey)) // Hash it

// Save to DB:
db.Create(&APIKey{
    Name: "default",
    Hash: hex.EncodeToString(hash[:]), // ONLY the hash!
    CreatedAt: time.Now(),
})

// Show to user:
fmt.Println("API Key:", apiKey) // Show ONLY ONCE!
```

## How is it Validated? ✅

```go
// User sends: Authorization: Bearer dpl_8f3d2a9b5c...

func validateAPIKey(providedKey string) bool {
    // Hash the provided key
    hash := sha256.Sum256([]byte(providedKey))
    hashStr := hex.EncodeToString(hash[:])

    // Compare with DB
    var apiKey APIKey
    result := db.Where("hash = ?", hashStr).First(&apiKey)

    return result.Error == nil
}
```

## Difference from JWT 🤔

**JWT:**

```
eyJhbGciOiJIUzI1NiIs...  // Contains data (user_id, expiry)
↓
Can be decoded
Needs secret to verify
Has expiration
```

**API Key:**

```
dpl_8f3d2a9b5c7e1f4a6d8b3c5e7f9a2d4e6b8c1d3f5a7b9c2e4f6a8d1b3c5e7f9
↓
Just random string
No secret needed
Never expires (unless revoked)
```

## Complete Implementation 🛠️

```go
// internal/auth/apikey/apikey.go

package apikey

import (
    "crypto/rand"
    "crypto/sha256"
    "encoding/hex"
    "fmt"
)

const PREFIX = "dpl_"
const KEY_LENGTH = 32 // bytes

func Generate() string {
    bytes := make([]byte, KEY_LENGTH)
    if _, err := rand.Read(bytes); err != nil {
        panic(err) // should never happen
    }
    return PREFIX + hex.EncodeToString(bytes)
}

func Hash(key string) string {
    hash := sha256.Sum256([]byte(key))
    return hex.EncodeToString(hash[:])
}

func Validate(key string, hash string) bool {
    return Hash(key) == hash
}
```

## Database Schema 📊

```sql
CREATE TABLE api_keys (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    hash TEXT NOT NULL UNIQUE,
    last_used_at TIMESTAMP,
    created_at TIMESTAMP NOT NULL,
    revoked_at TIMESTAMP
);

-- Index for fast lookups
CREATE INDEX idx_api_keys_hash ON api_keys(hash);
```

## During Server Install 🚀

```go
// scripts/install.sh calls:
func setupServer() {
    // Generate master API key
    apiKey := apikey.Generate()
    hash := apikey.Hash(apiKey)

    // Save to DB
    db.Create(&APIKey{
        Name: "master",
        Hash: hash,
    })

    // Show to user ONCE
    fmt.Printf("🔑 API Key: %s\n", apiKey)
    fmt.Println("Save this key! It won't be shown again.")
}
```

## Security Best Practices 🔒

1. **Length**: At least 32 bytes (256 bits)
2. **Crypto Random**: Never use math/rand!
3. **Hash Storage**: SHA256 or better
4. **Rate Limiting**: Max 5 failed attempts
5. **Audit Log**: Log every API call

## Why No JWT Secret?

- API Keys are **self-contained**
- No shared secret needed
- Simpler to implement
- Standard for CLI tools (GitHub, Stripe, etc.)

Makes sense? Much simpler than JWT! 😄

