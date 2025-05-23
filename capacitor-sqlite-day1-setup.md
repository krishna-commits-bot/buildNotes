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

## ❓ Developer Questions & Detailed Answers (50 Questions)

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

Use DB tools (like DB Browser for SQLite) or run in a Node.js test environment with mock data.

### 12. What is the best practice to separate read and write queries?

Use service-layer separation and transaction management. Group reads and writes to avoid conflicts.

### 13. Can mock data be reloaded/reset dynamically during development?

Yes, through scripts that drop and re-seed tables with sample data.

### 14. How large can the SQLite DB grow in a typical mobile environment?

It can handle gigabytes of data, but performance degrades beyond a few hundred MBs without indexing.

### 15. Does SQLite support JSON or unstructured data types?

Yes, via JSON1 extension, allowing storage and querying of JSON data.

### 16. What indexing strategies are supported?

B-tree indexing by default. Support for single, compound, and partial indexes.

### 17. How do we ensure ACID compliance with Capacitor’s plugin?

SQLite is ACID-compliant by design. Use transactions correctly via plugin methods.

### 18. What are the limitations of @capacitor-community/sqlite?

Not as mature as native implementations. Some advanced SQLite features may be unsupported.

### 19. Are there any tools for inspecting the SQLite DB on a real device?

Use Android Studio Device File Explorer or Xcode. You can also export the DB and inspect it with DB tools.

### 20. How to sync SQLite DB with remote APIs?

Use background sync jobs, REST APIs, and local change-tracking logic.

### 21. What is the best way to store binary data (e.g., images) in SQLite?

Store file paths in the DB, not binary blobs. This reduces DB bloat.

### 22. Are transactions supported natively in the plugin?

Yes, the plugin exposes transaction methods for batch inserts/updates.

### 23. How to handle rollback in case of failures?

Use transaction blocks. If any query fails, call rollback().

### 24. How to fetch large datasets without performance hits?

Use pagination and limit queries. Load only necessary fields.

### 25. Can SQLite queries be parameterized?

Yes, prepared statements can be used via plugin methods to avoid SQL injection.

### 26. What are the best practices for structuring multiple tables?

Normalize where necessary, use clear naming, index foreign keys, and design for extensibility.

### 27. How to handle nullable fields in SQLite schema?

Define fields without NOT NULL, and use NULL values explicitly.

### 28. How to integrate foreign keys and cascading deletes?

Use `ON DELETE CASCADE` and enable foreign key support using PRAGMA.

### 29. Is full-text search supported in SQLite?

Yes, using FTS5 extension, which allows full-text indexing and querying.

### 30. How to migrate user data between app versions?

Version control your schema and apply migrations conditionally during app start.

### 31. Is there a limit to the number of rows or tables?

SQLite supports millions of rows and thousands of tables, but performance depends on hardware.

### 32. How to manage SQLite DB across different platforms (iOS/Android)?

Use Capacitor plugins for platform-specific behavior. Maintain a shared schema and test both builds.

### 33. How to handle data conflicts during sync or updates?

Use conflict resolution policies (last-write wins, merge logic, or manual review).

### 34. Can we define triggers inside SQLite using Capacitor?

Yes, SQLite supports triggers; define them in schema migration scripts.

### 35. How to encrypt sensitive data in the database?

Use SQLCipher or encrypt at the application level before storing.

### 36. How does the performance degrade with 100k+ rows?

Without indexing, query time increases. Use indexes and optimize queries.

### 37. Can we run complex JOIN queries effectively?

Yes, SQLite handles joins well if indexes are defined properly.

### 38. What are common pitfalls in using SQLite in mobile apps?

Schema changes without migration, write-locks, DB corruption on crashes.

### 39. How to implement pagination for large result sets?

Use LIMIT and OFFSET in queries. Cache or lazy-load results.

### 40. How to debug SQLite issues on real devices?

Export the DB file, enable debug logs in app, use native tools for inspection.

### 41. How do different mobile OS versions affect SQLite behavior?

SQLite is bundled with the app, so behavior is mostly consistent unless plugin API changes.

### 42. Can SQLite database be bundled with the app initially?

Yes, pre-populate and copy DB file on first app launch.

### 43. What happens on app uninstall and reinstall?

All local data, including SQLite, is deleted unless backed up externally.

### 44. How to ensure cross-platform data consistency?

Use shared schema, rigorous testing, and common data validation logic.

### 45. Is lazy loading of data possible with SQLite?

Yes, by using paginated queries or loading chunks of data on demand.

### 46. How does SQLite behave in offline mode with sync capability?

Works offline; sync logic must track local changes and push to server when online.

### 47. What plugin APIs are exposed for backup/restore?

Capacitor plugin provides export/import and filesystem access to handle DB backup/restore.

### 48. Can a corrupted DB be detected and repaired automatically?

Detection is manual, repair is limited. Maintain backups and implement integrity checks.

### 49. How often should we vacuum or optimize the DB?

Periodically after many deletes. Use VACUUM to rebuild and optimize the DB.

### 50. What are best practices for writing safe and efficient SQL in mobile apps?

Use parameterized queries, proper indexing, normalized schema, and regular profiling.
