---
name: zerohelper
description: >
  Use this skill whenever the user works with the @onurege3467/zerohelper npm package.
  Triggers on: "zerohelper", "zero helper", "ZeroHelper", "zpack", "toon database", "zero init",
  "zero cli", "database.createDatabase", "functions.ai_module", "functions.crypto_module",
  "functions.security_module", "ZeroWorker", "MigrationManager", "DataSeeder", or any mention of
  ZeroHelper's database adapters (zpack, toon, json, sqlite, mysql, postgres, mongodb, redis via zerohelper).
  Always use this skill when writing code that imports from '@onurege3467/zerohelper', even if the user
  doesn't explicitly say "zerohelper" — infer from the import path.
---

# ZeroHelper Skill

**Package:** `@onurege3467/zerohelper` (v11.x)  
**Install:** `npm install @onurege3467/zerohelper`  
**CLI:** `npx zero <command>`

TypeScript-native Node.js utility + multi-database framework. Two main namespaces: `database` and `functions`.

---

## Import Pattern

```typescript
import { database, functions } from '@onurege3467/zerohelper';
```

---

## 1. DATABASE

### Adapter Selection Guide

| Adapter | `adapter` key | Best For |
|---------|--------------|----------|
| ZPack (binary) | `'zpack'` | High-volume logging, audit trails, archival |
| TOON | `'toon'` | AI/LLM apps, config files, token-efficient storage |
| JSON | `'json'` | Prototyping, small projects |
| SQLite | `'sqlite'` | Desktop apps, single-user |
| MySQL | `'mysql'` | Web apps (with promise-mysql) |
| PostgreSQL | `'postgres'` | Enterprise systems |
| MongoDB | `'mongodb'` | Flexible document schemas |
| Redis | `'redis'` | Caching, sessions, rate limiting |

### createDatabase

```typescript
const db = database.createDatabase({ adapter: 'zpack', config: { path: './data.zpack' } });
```

### IDatabase — Universal API (all adapters)

```typescript
// INSERT
const id = await db.insert('table', { field: value });
const count = await db.bulkInsert('table', [row1, row2]);

// SELECT
const rows = await db.select<T>('table', { where: 'clause' });
const row  = await db.selectOne<T>('table', { id: 1 });

// UPDATE / DELETE
await db.update('table', { field: newVal }, { id: 1 });
await db.delete('table', { id: 1 });

// UPSERT
await db.set('settings', { value: 'dark' }, { key: 'theme' });

// ATOMIC COUNTERS
await db.increment('wallets', { balance: 100 }, { user_id: 1 });
await db.decrement('inventory', { stock: 1 }, { sku: 'PRO-123' });
```

### Lifecycle Hooks

```typescript
db.on('beforeInsert', (table, data) => { data.updated_at = Date.now(); });
db.on('afterUpdate',  (table, result) => { console.log(result.affected); });
// Events: beforeInsert, afterInsert, beforeUpdate, afterUpdate, beforeDelete, afterDelete
```

### Caching (LRU or Redis)

```typescript
const db = database.createDatabase({
  adapter: 'mysql',
  config: {
    host: 'localhost',
    cache: { type: 'redis', host: '127.0.0.1', ttl: 300000 }
    // type: 'memory' for LRU
  }
});
```

### Telemetry

```typescript
const metrics = db.getMetrics();
// metrics.database.averageDuration
// metrics.cache.ratio
```

### ZPack — Binary Adapter

```typescript
const db = database.createDatabase({
  adapter: 'zpack',
  config: {
    path: './events.zpack',
    indexFields: { events: ['event_type', 'user_id'] },
    autoFlush: true
  }
});
await db.vacuum(); // defragment — run periodically
```
⚠️ ZPack is single-writer. Don't use for multi-threaded concurrent writes.

### TOON — AI-Optimized Format

```typescript
const db = database.createDatabase({
  adapter: 'toon',
  config: { path: './ai-data.toon', saveInterval: 1000 }
});
```

TOON syntax saves ~75% tokens vs JSON. Also usable as a module:

```typescript
const str = functions.toon_module.stringify({ a: 1 });
const obj = functions.toon_module.parse(str);
```

### MigrationManager

```typescript
const migration = new database.MigrationManager(db);
migration.createMigration('add_profile_pictures');
await migration.migrate();
// CLI: npx zero migrate | npx zero migration:rollback --steps 1
```

### DataSeeder

```typescript
const seeder = new database.DataSeeder(db);
await seeder.seed('users', 100, {
  email:    { type: 'email' },
  age:      { type: 'number', min: 18, max: 65 },
  isActive: { type: 'boolean' }
});
// Field types: string, number, email, boolean, date, id, pick
```

---

## 2. FUNCTIONS

### 2a. AI Module (`functions.ai_module`)

For LLM/AI utility work — token counting, prompt building, cost estimation.

```typescript
// Token ops
functions.ai_module.estimateTokens(text);
functions.ai_module.truncateToTokenLimit(text, 1000);
functions.ai_module.splitByTokenLimit(text, 4000);
functions.ai_module.isTokenLimitExceeded(prompt, 4000);
functions.ai_module.truncatePromptByPercentage(prompt, 50);

// Prompt building
functions.ai_module.formatChatMessage('user', 'Hello');
functions.ai_module.createFewShotPrompt([{ input: '2+2?', output: '4' }], '3+3?');
functions.ai_module.mergeSystemAndUser('You are helpful', 'What is weather?');

// Context management
functions.ai_module.fitMessagesToContext(messages, 8000);
functions.ai_module.compressConversationHistory(messages, 4);

// Cost estimation
const cost = functions.ai_module.estimateConversationCost(messages, 'gpt-4');
// cost.totalCost — USD
functions.ai_module.getModelTokenLimit('gpt-4o');      // 128000
functions.ai_module.getModelPricing('claude-3-sonnet');

// Output parsing
functions.ai_module.extractJSONFromMarkdown('```json\n{}\n```');
functions.ai_module.parseStreamingChunk(rawChunk);
```

### 2b. Crypto Module (`functions.crypto_module`)

```typescript
// Password
const hash = functions.crypto_module.hashPassword('pass123');
functions.crypto_module.verifyPassword('pass123', hash);   // boolean
functions.crypto_module.isPasswordStrong('MyP@ss123');
functions.crypto_module.generateSalt();

// AES-256-CBC encryption
const { encryptedText, iv } = functions.crypto_module.encryptText('secret', 'key');
functions.crypto_module.decryptText(encryptedText, 'key', iv);

// JWT
const token   = functions.crypto_module.generateJWT({ userId: 1 }, 'secret');
const payload = functions.crypto_module.verifyJWT(token, 'secret');

// UUID
functions.crypto_module.validateUUID('550e8400-...');
```

### 2c. Security Module (`functions.security_module`)

```typescript
// Memory rate limit (single instance)
const result = await functions.security_module.checkRateLimit('user:1', {
  limit: 10, window: 60, storage: 'memory'
});
result.allowed; // boolean

// Redis rate limit (distributed)
const result = await functions.security_module.checkRateLimit(`api:${ip}`, {
  limit: 1000, window: 3600, storage: 'redis', redisClient: redisDb
});
```

### 2d. Validation Module (`functions.validation_module`)

```typescript
// Schema validation
const schema = {
  email:    { required: true,  type: 'string', pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/ },
  age:      { required: false, type: 'number', min: 18, max: 120 },
  password: { required: true,  type: 'string', minLength: 8 }
};
const { isValid, errors } = functions.validation_module.validateSchema(data, schema);

// Sanitization
const clean = functions.validation_module.sanitizeInput(userInput, {
  trim: true, removeHTML: true, escape: true
});

// Quick checks
functions.validation_module.isEmail('x@y.com');
functions.validation_module.isPhone('+1234567890');
functions.validation_module.isURL('https://example.com');
functions.validation_module.sanitizeHTML('<script>xss</script>');
functions.validation_module.validateCreditCard('4111111111111111');
```

### 2e. Logger Module (`functions.logger_module`)

```typescript
const logger = functions.logger_module.createLogger({
  level: 'info',        // 'error' | 'warn' | 'info' | 'debug'
  enableColors: true,
  enableTimestamp: true,
  logFile: './app.log'  // null to disable file logging
});

logger.info('User logged in', { userId: 123 });
logger.warn('Rate limit approaching');
logger.error('DB failed', { error: err.message });
logger.debug('Cache hit', { key: 'user:123' });

// Quick static methods
functions.logger_module.info('msg');
functions.logger_module.error('msg');
```

### 2f. Random Module (`functions.random_module`)

```typescript
functions.random_module.makeUniqueId();
functions.random_module.randomHex();
functions.random_module.randomEmoji();
functions.random_module.randomText(16);
functions.random_module.randomNumber(1, 100);
functions.random_module.randomFloat(0.0, 1.0);
functions.random_module.randomArray([1, 2, 3]); // picks one
```

### 2g. String Module (`functions.string_module`)

```typescript
functions.string_module.generateSlug('ZeroHelper: The Best!'); // 'zerohelper-the-best'
functions.string_module.titleCase('hello world');
functions.string_module.wordCount('hello world'); // 2
functions.string_module.generateRandomString(12);
```

### 2h. Array Module (`functions.array_module`)

```typescript
functions.array_module.shuffleArray([1, 2, 3, 4, 5]);
functions.array_module.groupBy(users, 'role');
functions.array_module.pluck(users, 'email');
functions.array_module.sortBy(users, 'name');
functions.array_module.flattenArray([[1, 2], [3, 4]]);
functions.array_module.removeFalsyValues([0, null, 1, '', 2]);
```

### 2i. Math Module (`functions.math_module`)

```typescript
const data = [10, 2, 38, 23, 21];
functions.math_module.mean(data);
functions.math_module.median(data);
functions.math_module.variance(data);
functions.math_module.standardDeviation(data);
functions.math_module.sum(data);
functions.math_module.max(data);
functions.math_module.min(data);
functions.math_module.range(1, 10);
functions.math_module.isPrime(13);
functions.math_module.factorial(5);
functions.math_module.combination(10, 3);
functions.math_module.permutation(10, 3);
```

### 2j. Date Module (`functions.date_module`)

```typescript
functions.date_module.formatDate(new Date(), 'YYYY-MM-DD');
functions.date_module.addDays(new Date(), 5);
functions.date_module.subtractDays(new Date(), 3);
functions.date_module.dateDifference(date1, date2); // ms
```

### 2k. HTTP Module (`functions.http_module`)

```typescript
const data     = await functions.http_module.fetchData('https://api.example.com/data');
const response = await functions.http_module.postData('https://api.example.com/post', { key: 'val' });
```

### 2l. Object Module (`functions.object_module`)

```typescript
functions.object_module.deepMerge(obj1, obj2);
functions.object_module.filterObjectByKey(obj, ['name', 'email']);
```

### 2m. ZeroWorker (`functions.worker_module`)

CPU-bound tasks in a Worker Thread — non-blocking.

```typescript
const result = await functions.worker_module.runAsyncTask(
  "(data) => data.map(x => x * 2)",
  [1, 2, 3]
);
```

---

## 3. ZERO-CLI

```bash
npx zero init                          # scaffold zero.config.ts

# DB
npx zero db:test
npx zero db:stats
npx zero db:backup [--output ./dir]
npx zero db:restore ./backup.json
npx zero db:seed --table users --count 100

# Migrations
npx zero migrate
npx zero make:migration create_users_table
npx zero migration:rollback --steps 1
npx zero migration:status

# Cache
npx zero cache:clear
npx zero cache:stats

# Export / Import
npx zero db:export --table users --format json
npx zero db:export --table users --format csv --output ./users.csv
npx zero db:import ./users.csv --table users --format csv

# ZPack
npx zero zpack:vacuum ./data.zpack

# REPL
npx zero repl
# REPL commands: select <table> | count <table> | .stats | .metrics | .exit
```

Global flag: `-c, --config <path>` (default: `zero.config.ts`)

---

## 4. COMMON PATTERNS

### Full-stack API with Postgres + Redis cache + JWT

```typescript
import { database, functions } from '@onurege3467/zerohelper';

const db = database.createDatabase({
  adapter: 'postgres',
  config: {
    host: 'localhost', user: 'admin', password: 'secret', database: 'app',
    cache: { type: 'redis', host: '127.0.0.1', ttl: 300000 }
  }
});

async function register(email: string, password: string) {
  const { isValid, errors } = functions.validation_module.validateSchema(
    { email, password },
    { email: { required: true, type: 'string', pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/ },
      password: { required: true, type: 'string', minLength: 8 } }
  );
  if (!isValid) throw new Error(errors.join(', '));

  const hash = functions.crypto_module.hashPassword(password);
  const id   = await db.insert('users', { email, password: hash });
  const token = functions.crypto_module.generateJWT({ userId: id }, process.env.JWT_SECRET!);
  return token;
}
```

### AI App with TOON + token management

```typescript
const toonDb = database.createDatabase({ adapter: 'toon', config: { path: './prompts.toon' } });

async function prepareContext(messages: any[]) {
  const fitted    = functions.ai_module.fitMessagesToContext(messages, 8000);
  const compressed = functions.ai_module.compressConversationHistory(fitted, 4);
  const cost      = functions.ai_module.estimateConversationCost(compressed, 'gpt-4');

  await toonDb.insert('sessions', { messages: compressed, estimatedCost: cost.totalCost });
  return compressed;
}
```

### Distributed Rate Limiting

```typescript
const redisDb = database.createDatabase({ adapter: 'redis', config: { host: '127.0.0.1' } });

async function apiGuard(ip: string) {
  const { allowed } = await functions.security_module.checkRateLimit(`api:${ip}`, {
    limit: 100, window: 60, storage: 'redis', redisClient: redisDb
  });
  if (!allowed) throw new Error('429 Rate limit exceeded');
}
```

---

## 5. NOTES & GOTCHAS

- **ZPack** — single-writer only. Use `vacuum()` periodically to defragment.
- **TOON** — saves ~75% tokens vs JSON. Ideal for LLM context storage.
- **Auto schema** — MySQL/PostgreSQL adapters auto-create tables and `ALTER TABLE` on new keys.
- **Transactions** — only supported on SQL adapters (MySQL, PostgreSQL, SQLite).
- **Cache invalidation** — automatic on writes when caching is enabled.
- **Worker functions** — must be serializable strings (no closures over outer scope).
- **hashPassword** uses bcrypt (sync) — avoid calling in hot loops; use workers for batch hashing.
