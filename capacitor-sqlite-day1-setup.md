# Day 1 – Setup SQLite Database with Capacitor

## Goals:
- Setup local SQLite database using Capacitor
- Define the initial database schema
- Insert mock data for testing purposes

---

## 🧱 SQLite + Capacitor Setup

Capacitor is a cross-platform native runtime by Ionic. It allows us to use native functionality in web-based mobile apps. For storing structured data, Capacitor supports SQLite through plugins.

### Why SQLite?

- Lightweight and fast
- Works well for mobile applications
- Easy to integrate with Capacitor using `@capacitor-community/sqlite`

---

## 🗂️ Initial Schema Definition

Here is an example schema to track job applications (used in job tracking apps):

```sql
CREATE TABLE job_applications (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    company_name TEXT NOT NULL,
    job_title TEXT NOT NULL,
    status TEXT NOT NULL, -- e.g., 'Applied', 'Interviewing', 'Offer', 'Rejected'
    application_date TEXT,
    notes TEXT
);
```

---

## 🔄 Insert Mock Data

We insert a few records to test our queries and structure:

```sql
INSERT INTO job_applications (company_name, job_title, status, application_date, notes)
VALUES ('OpenAI', 'Software Engineer', 'Applied', '2025-05-21', 'Excited about the role.');

INSERT INTO job_applications (company_name, job_title, status, application_date, notes)
VALUES ('Google', 'Backend Developer', 'Interviewing', '2025-05-10', 'Technical round scheduled.');

INSERT INTO job_applications (company_name, job_title, status, application_date, notes)
VALUES ('Meta', 'Frontend Engineer', 'Rejected', '2025-04-29', 'Rejection email received.');
```

---

## ❓Questions & Detailed Answers

### 1. How does Capacitor handle native SQLite operations under the hood?

Capacitor uses a JavaScript-to-native bridge. The `@capacitor-community/sqlite` plugin wraps native SQLite implementations for Android (Java/Kotlin) and iOS (Swift/Obj-C). When you call a JS method like `createConnection`, it's translated into native function calls internally via Capacitor’s plugin system.

📦 **Under the Hood:**
- JavaScript calls → Capacitor Plugin Bridge → Native SQLite API
- Platform code uses `sqlite3` (C library) via native bindings

🔧 **Example (JS):**
```js
import { CapacitorSQLite } from '@capacitor-community/sqlite';
const sqlite = CapacitorSQLite;

const db = await sqlite.createConnection({ database: 'job_tracker' });
await db.open();
await db.execute("CREATE TABLE IF NOT EXISTS jobs (id INTEGER PRIMARY KEY, title TEXT)");
```

For More Reference, Visit [Capacitor SQLite GitHub](https://github.com/capacitor-community/sqlite)

### 2. Is data persisted across app restarts?

✅ Yes. SQLite databases are saved in the app's sandboxed file system, typically under `/data/data/{your.app.package}/databases/` on Android or the Documents directory on iOS. This persists through restarts unless the app is uninstalled.

### 3. How does performance compare with alternatives like IndexedDB or Realm?

| Feature          | SQLite                        | IndexedDB                  | Realm                     |
|------------------|-------------------------------|----------------------------|---------------------------|
| **Speed**        | ✅ Fast for relational data    | ⚠️ Slower with structure    | ✅ Fast with sync          |
| **Sync Support** | Manual                        | Browser Only               | ✅ Built-in Sync           |
| **Structured Data** | ✅ Best fit               | Not ideal                  | Mixed                     |

📘 **SQLite is a great fit when:**

- Your data is relational (tables, foreign keys)
- You need performant local storage for 1000s+ rows

🛠️ **Use Realm if:**

- You need real-time sync out of the box

### 4. Can the database be backed up or exported?

Yes. You can export the DB file using:

- Capacitor Filesystem API
- Plugin methods like `exportToJson()` or `copyFromAssets()`

🧪 **Example:**

```js
await sqlite.exportToJson({ database: 'job_tracker', jsonexportmode: 'full' });


### 5. How to reset or wipe the local SQLite DB?

You can either:

- Drop all tables
- Delete the entire `.db` file
- Call plugin’s delete method

🔧 **Example:**

```js
await sqlite.deleteDatabase({ database: 'job_tracker' });
```
**Or drop tables:**
```js
await db.execute("DROP TABLE IF EXISTS job_applications");
```

### 6. What happens if schema changes are introduced?

Schema changes without migrations = ⚠️ crash or inconsistent state.

🧠 **Best Practice:**

- Add a `schema_version` table
- On app start, check version and run migrations accordingly

🔧 **Migration Example:**

```js
await db.execute("ALTER TABLE job_applications ADD COLUMN location TEXT;");
```

### 7. How to version the database schema?

Create a metadata or version table:

```sql
CREATE TABLE IF NOT EXISTS schema_version (
  version INTEGER
);
```
**🔁 On startup:**
```js
const res = await db.query("SELECT version FROM schema_version");
if (res.values[0].version < 2) {
  await db.execute("ALTER TABLE job_applications ADD COLUMN remote BOOLEAN;");
  await db.execute("UPDATE schema_version SET version = 2");
}

```

### 8. Is there support for relational integrity (foreign keys)?

✅ Yes, SQLite supports foreign keys. However, they are disabled by default.

🔧 **Enable foreign keys:**

```js
await db.execute("PRAGMA foreign_keys = ON;");
```

**🔧 Example with foreign key:**

```sql
CREATE TABLE departments (
  id INTEGER PRIMARY KEY,
  name TEXT
);

CREATE TABLE employees (
  id INTEGER PRIMARY KEY,
  name TEXT,
  department_id INTEGER,
  FOREIGN KEY(department_id) REFERENCES departments(id)
);

```

### 9. How secure is the local SQLite data?

⚠️ By default, SQLite databases are not encrypted.

🔐 **For sensitive data:**

- Use SQLCipher (an encrypted extension of SQLite)
- Encrypt/decrypt the DB using native code or supported wrappers

🔧 **Options:**

- Use SQLCipher build with Capacitor SQLite
- Or encrypt specific fields manually

 [📘SQLCipher Integration Guide](https://www.zetetic.net/sqlcipher/)

### 10. Can SQLite handle concurrent reads/writes in a mobile app?

✅ SQLite allows:

- Multiple concurrent reads
- Only one write at a time (due to write-lock)

⚠️ Long write transactions can block reads, so:

- Use transactions efficiently
- Avoid long-running queries
- Consider background threads (native) or batching operations

🔧 **Example (Safe transaction):**

```js
await db.execute("BEGIN TRANSACTION;");
await db.execute("INSERT INTO jobs (title) VALUES ('Dev')");
await db.execute("COMMIT;");
```

[SQLite Concurrency](https://sqlite.org/concurrent.html)

### 11. How to test SQL queries locally without running the full app?

You can use tools like **DB Browser for SQLite** to test queries visually or set up a **Node.js environment** with SQLite3 package and mock data to validate logic before integrating.

🔧 **Node.js Example:**
```js
const sqlite3 = require('sqlite3').verbose();
const db = new sqlite3.Database(':memory:');

db.serialize(() => {
  db.run("CREATE TABLE jobs (id INTEGER, title TEXT)");
  db.run("INSERT INTO jobs VALUES (1, 'Engineer')");
  db.each("SELECT * FROM jobs", (err, row) => console.log(row));
});
```

---

### 12. What is the best practice to separate read and write queries?

Use a **service layer** pattern to isolate read (SELECT) and write (INSERT, UPDATE, DELETE) operations.

✅ Benefits:
- Prevent read-write conflicts
- Easier caching and optimization
- Maintain clear data flow

---

### 13. Can mock data be reloaded/reset dynamically during development?

Yes. Write dev-only scripts or CLI commands that:
- Drop tables
- Recreate schema
- Seed with mock data

🔧 **Example:**
```js
await db.execute("DROP TABLE IF EXISTS jobs");
await db.execute("CREATE TABLE jobs (id INTEGER, title TEXT)");
await db.execute("INSERT INTO jobs (title) VALUES ('Dev'), ('QA'), ('PM')");
```

---

### 14. How large can the SQLite DB grow in a typical mobile environment?

SQLite can handle databases in **GBs**, but:

- Performance degrades beyond a few hundred MBs
- Use **indexes** and **normalized schema**
- Avoid unnecessary BLOBs

📘 [SQLite Limits](https://www.sqlite.org/limits.html)

---

### 15. Does SQLite support JSON or unstructured data types?

✅ Yes. Use the **JSON1 extension** to:
- Store JSON as TEXT
- Query JSON fields using functions like `json_extract`, `json_object`, etc.

🔧 **Example:**
```sql
SELECT json_extract(data, '$.name') FROM users WHERE id = 1;
```

---

### 16. What indexing strategies are supported?

SQLite supports:

- **Single-column index**
- **Multi-column (compound) index**
- **Partial indexes** (using WHERE clause)
- **Unique indexes**

🔧 **Example:**
```sql
CREATE INDEX idx_title ON jobs(title);
CREATE UNIQUE INDEX idx_unique_email ON users(email);
```

📘 [SQLite Indexes](https://sqlite.org/lang_createindex.html)

---

### 17. How do we ensure ACID compliance with Capacitor’s plugin?

SQLite is **ACID-compliant by default**. To ensure compliance:

- Always wrap related writes in `BEGIN TRANSACTION` and `COMMIT`
- Rollback on error

🔧 **Example:**
```js
await db.execute("BEGIN TRANSACTION;");
// multiple inserts/updates
await db.execute("COMMIT;");
```

---

### 18. What are the limitations of @capacitor-community/sqlite?

- Some advanced SQLite features (e.g., custom collations, triggers) may not be supported
- May require native setup for encryption or file access
- Slightly larger bundle size

📘 [Plugin Issues Tracker](https://github.com/capacitor-community/sqlite/issues)

---

### 19. Are there any tools for inspecting the SQLite DB on a real device?

✅ Yes:

- **Android:** Use Android Studio > Device File Explorer
- **iOS:** Use Xcode > Devices > App container download
- **Both:** Export `.db` file via Capacitor Filesystem

🔧 **DB Tools:** [DB Browser for SQLite](https://sqlitebrowser.org/), DBeaver

---

### 20. How to sync SQLite DB with remote APIs?

Use:

1. REST APIs for CRUD operations
2. Local change tracking (timestamp or dirty flags)
3. Background job processing (e.g., WorkManager/Background Fetch)
4. Conflict resolution strategies

🔧 **Example Sync Flow:**
```txt
[Local Change] → [Flag as dirty] → [Background Sync Job] → [Remote API] → [Clear Flag]
```

### 21. What is the best way to store binary data (e.g., images) in SQLite?
Storing binary blobs directly in SQLite is possible but not ideal for performance or scalability. The best practice is to store the file path or URI in the database and keep the binary data (like images) in the device's file system.

**Example:**
```sql
CREATE TABLE photos (
    id INTEGER PRIMARY KEY,
    title TEXT,
    file_path TEXT
);
```

### 22. Are transactions supported natively in the plugin?
Yes, the `@capacitor-community/sqlite` plugin supports transactions. Transactions allow multiple operations to be treated as a single unit of work.

**Example:**
```ts
await db.execute('BEGIN TRANSACTION;');
try {
  await db.run('INSERT INTO users(name) VALUES (?)', ['Krishna']);
  await db.run('INSERT INTO logs(action) VALUES (?)', ['user added']);
  await db.execute('COMMIT;');
} catch (error) {
  await db.execute('ROLLBACK;');
}
```

### 23. How to handle rollback in case of failures?
You handle rollback by wrapping your operations in a transaction. If any query fails, call `ROLLBACK` to undo changes.

### 24. How to fetch large datasets without performance hits?
Use SQL pagination (`LIMIT` and `OFFSET`) and only fetch the fields you need. Avoid `SELECT *` for large tables.

**Example:**
```sql
SELECT id, title FROM jobs ORDER BY applied_on DESC LIMIT 20 OFFSET 40;
```

### 25. Can SQLite queries be parameterized?
Yes, parameterized queries (prepared statements) help prevent SQL injection and improve performance.

**Example:**
```ts
await db.run('SELECT * FROM users WHERE email = ?', [userEmail]);
```

### 26. What are the best practices for structuring multiple tables?
- Normalize your schema to reduce redundancy.
- Use intuitive and consistent naming.
- Index foreign keys and columns frequently used in WHERE clauses.
- Design tables for expected app workflows.

### 27. How to handle nullable fields in SQLite schema?
Declare the field without the `NOT NULL` constraint. Store `NULL` explicitly.

**Example:**
```sql
CREATE TABLE tasks (
    id INTEGER PRIMARY KEY,
    description TEXT,
    due_date TEXT NULL
);
```

### 28. How to integrate foreign keys and cascading deletes?
Use the `ON DELETE CASCADE` clause in foreign key definitions and ensure foreign keys are enabled:

**Example:**
```sql
PRAGMA foreign_keys = ON;

CREATE TABLE parent (
    id INTEGER PRIMARY KEY
);

CREATE TABLE child (
    id INTEGER PRIMARY KEY,
    parent_id INTEGER,
    FOREIGN KEY(parent_id) REFERENCES parent(id) ON DELETE CASCADE
);
```

### 29. Is full-text search supported in SQLite?
Yes, SQLite provides full-text search through the FTS5 extension. It enables indexing and efficient querying of text content.

**Example:**
```sql
CREATE VIRTUAL TABLE articles USING fts5(title, body);
SELECT * FROM articles WHERE body MATCH 'Capacitor';
```

### 30. How to migrate user data between app versions?
Keep a `version` number for your schema. On app start, compare the current version with the DB version, and apply migration scripts accordingly.

**Example:**
```ts
if (dbVersion < 2) {
  await db.execute('ALTER TABLE users ADD COLUMN last_login TEXT;');
  await setDbVersion(2);
}
```

### 31. Is there a limit to the number of rows or tables?
SQLite supports millions of rows and thousands of tables, but performance depends on device hardware and indexing strategies. Efficient indexing and careful schema design are key to scaling.

### 32. How to manage SQLite DB across different platforms (iOS/Android)?
Use the `@capacitor-community/sqlite` plugin which handles platform differences. Keep the schema consistent across platforms, and test on both to catch any discrepancies.

### 33. How to handle data conflicts during sync or updates?
Implement conflict resolution strategies such as:
- Last-write-wins.
- Merging based on timestamps.
- Prompting the user to resolve conflicts manually.

### 34. Can we define triggers inside SQLite using Capacitor?
Yes. Triggers are part of SQLite's core functionality. You can include them in schema setup scripts:

```sql
CREATE TRIGGER update_timestamp
AFTER UPDATE ON users
FOR EACH ROW
BEGIN
  UPDATE users SET updated_at = CURRENT_TIMESTAMP WHERE id = OLD.id;
END;
```

### 35. How to encrypt sensitive data in the database?
Use SQLCipher for database-level encryption.  
Alternatively, encrypt data before storing using JavaScript libraries (e.g., crypto-js).

### 36. How does the performance degrade with 100k+ rows?
Performance may degrade due to:
- Lack of indexes.
- Large joins or subqueries.

Use `LIMIT`, `OFFSET`, and indexing to maintain performance.

### 37. Can we run complex JOIN queries effectively?
Yes. SQLite supports multi-table joins. Ensure that joined fields are indexed to avoid slowdowns.

```sql
SELECT orders.id, customers.name
FROM orders
JOIN customers ON orders.customer_id = customers.id;
```

### 38. What are common pitfalls in using SQLite in mobile apps?
- Missing migrations.
- Write locks causing crashes.
- DB corruption on improper app closure.
- Lack of schema versioning.

### 39. How to implement pagination for large result sets?
Use SQL `LIMIT` and `OFFSET`:

```sql
SELECT * FROM jobs LIMIT 20 OFFSET 40;
```

Implement lazy loading or infinite scrolling for better UX.

### 40. How to debug SQLite issues on real devices?
- Export the DB file and inspect with tools like DB Browser for SQLite.
- Enable verbose logging in app code.
- Use Android Studio’s Device File Explorer or Xcode to pull files.

# Other Questions : 

### 41. How do different mobile OS versions affect SQLite behavior?
SQLite is bundled with the app via the plugin, ensuring consistent behavior across OS versions. However, updates to the plugin or OS-level file access policies may affect behavior. Regular testing on multiple OS versions is recommended.

### 42. Can SQLite database be bundled with the app initially?
Yes. You can pre-populate a database and bundle it with your app. On the first app launch, copy the database to the appropriate app directory using the plugin’s import functionality.

### 43. What happens on app uninstall and reinstall?
When an app is uninstalled, all local data including the SQLite database is deleted. To preserve data across reinstalls, store backups externally (e.g., cloud storage or exported files).

### 44. How to ensure cross-platform data consistency?
- Use a shared database schema across all platforms.
- Apply consistent data validation logic.
- Perform rigorous cross-platform testing.
- Use the same plugin version on all platforms.

### 45. Is lazy loading of data possible with SQLite?
Yes. Implement lazy loading using:
- `LIMIT` and `OFFSET` for paginated queries.
- Chunked data retrieval as the user scrolls or interacts with the app.

```sql
SELECT * FROM items LIMIT 20 OFFSET 40;
```

### 46. How does SQLite behave in offline mode with sync capability?
SQLite works fully offline. To sync with a server:
- Track local changes (e.g., timestamps or version fields).
- Implement a queue or sync strategy to push changes when back online.

### 47. What plugin APIs are exposed for backup/restore?
The `@capacitor-community/sqlite` plugin provides:
- `exportToJson` and `importFromJson` methods.
- File system access to export/import the `.db` file manually.

### 48. Can a corrupted DB be detected and repaired automatically?
Detection must be handled manually using checks like:
- `PRAGMA integrity_check;`
- Fallback strategies and logging.

Automatic repair is limited; always maintain regular backups for recovery.

### 49. How often should we vacuum or optimize the DB?
Run `VACUUM`:
- After bulk deletions.
- Periodically in a maintenance routine.

It reclaims unused space and optimizes database performance.

### 50. What are best practices for writing safe and efficient SQL in mobile apps?
- Use **parameterized queries** to prevent SQL injection.
- Create **indexes** on frequently queried columns.
- Maintain a **normalized schema** to reduce redundancy.
- Profile queries during development to detect bottlenecks.