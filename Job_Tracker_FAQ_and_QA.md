
# Job Tracker App – Developer Q&A (SQLite + Capacitor + React)

This markdown file documents the key questions and answers that may arise during development of a **local-first**, **serverless**, **React + Capacitor** job tracking application, using **SQLite** and **Vector Embeddings**.

---

### ❓ How to manage and structure the local SQLite database in a Capacitor React app?

**Answer:**
Use the `@capacitor-community/sqlite` plugin to manage SQLite locally. Create a DB connection and structure your schema in an initialization script.

```bash
npm install @capacitor-community/sqlite
npx cap sync
```

```ts
// database.ts
import { CapacitorSQLite } from '@capacitor-community/sqlite';

export async function initDB() {
  const sqlite = new CapacitorSQLite();
  const db = await sqlite.createConnection('job_tracker_db', false, 'no-encryption', 1);
  await db.open();
  await db.execute(\`
    CREATE TABLE IF NOT EXISTS job_applications (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      company_name TEXT NOT NULL,
      job_title TEXT NOT NULL,
      location TEXT,
      applied_date TEXT,
      status TEXT,
      notes TEXT,
      created_at TEXT DEFAULT (datetime('now')),
      updated_at TEXT DEFAULT (datetime('now'))
    );
  \`);
  return db;
}
```

---

### ❓ How to fetch, add, update, and delete jobs from the database using React?

**Answer:**
Create custom hooks like `useJobs` to abstract database operations from UI components.

```ts
export function useJobs() {
  const [jobs, setJobs] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function loadJobs() {
      await initDB();
      const db = getDB();
      const res = await db.query('SELECT * FROM job_applications ORDER BY applied_date DESC');
      setJobs(res.values);
      setLoading(false);
    }
    loadJobs();
  }, []);

  // addJob, updateJob, deleteJob go here
}
```

---

### ❓ How to make the app work offline and fully serverless?

**Answer:**
Capacitor allows apps to use local device APIs and persist data offline. With SQLite and no dependency on remote servers, all data remains local. For AI features, embeddings can be stored locally once generated.

---

### ❓ How to integrate vector embeddings for semantic search?

**Answer:**
Use a service like OpenAI to generate embeddings and store them in the SQLite DB as JSON arrays.

```ts
import { OpenAIApi, Configuration } from 'openai';

const openai = new OpenAIApi(new Configuration({ apiKey: 'YOUR_KEY' }));

export async function getEmbedding(text) {
  const res = await openai.createEmbedding({
    model: 'text-embedding-ada-002',
    input: text
  });
  return res.data.data[0].embedding;
}
```

Then store it in your DB:

```ts
await db.run('INSERT INTO job_applications (...) VALUES (?, ?, ?, ?)', [..., JSON.stringify(embedding)]);
```

---

### ❓ How to implement semantic search using cosine similarity?

**Answer:**
After fetching all stored embeddings from the DB, compare them to the query embedding:

```ts
function cosineSimilarity(vecA, vecB) {
  const dot = vecA.reduce((sum, a, i) => sum + a * vecB[i], 0);
  const magA = Math.sqrt(vecA.reduce((sum, a) => sum + a * a, 0));
  const magB = Math.sqrt(vecB.reduce((sum, b) => sum + b * b, 0));
  return dot / (magA * magB);
}
```

---

### ❓ Can this system scale?

**Answer:**
Yes — for **personal/local use**, it’s efficient and instant. If scaling to multiple users, you may need to sync or switch to a remote backend.

---

### ❓ What benefits does using Capacitor bring here?

**Answer:**
Capacitor lets you write web-first code that can be deployed as a native app with full access to local storage, plugins, and device APIs — giving this job tracker app a “native” feel with local power.

---

### ❓ Can we show search results based on user-entered notes and job titles?

**Answer:**
Absolutely! That’s the purpose of embeddings — you can allow free-text search that intelligently matches relevant jobs even if keywords don’t exactly match.

---

> “When your job tracker understands your notes better than your boss — you're winning.”

