
# Job Tracker App – Dev Guide with a Dash of Fun

Welcome to the **Developer's Handbook** for the *serverless, locally working, AI-powered Job Tracker App* built with **React + Capacitor + SQLite + Vector Embeddings**!

Let’s dive in and build something awesome – one `.hook` at a time!

---

## ⚙️ Tech Stack Overview

| Area | Tool |
|------|------|
| Frontend | ReactJS + Capacitor |
| Database | SQLite (local, via Capacitor plugin) |
| AI/Vector | Vector Embeddings using OpenAI or local models |
| State | Custom React Hooks |
| Platform | Works offline, fully serverless |

---

## 1. **SQLite Schema & Initialization**

```bash
npm install @capacitor-community/sqlite
npx cap sync
```

### `database.ts`

```ts
import { CapacitorSQLite, SQLiteDBConnection } from '@capacitor-community/sqlite';

let db: SQLiteDBConnection | null = null;

export async function initDB() {
  const sqlite = new CapacitorSQLite();
  db = await sqlite.createConnection('job_tracker_db', false, 'no-encryption', 1);
  await db.open();

  const createTableQuery = \`
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
  \`;
  await db.execute(createTableQuery);
  return db;
}

export function getDB() {
  if (!db) throw new Error('DB not initialized. Call initDB first.');
  return db;
}
```

> "Because every great job app deserves a great DB!"

---

## 2. **React Hook: `useJobs`**

Meet your new best friend. It brings the jobs, adds new ones, updates, deletes — like a personal assistant but cooler.

```ts
export function useJobs() {
  const [jobs, setJobs] = useState<JobApplication[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function loadJobs() {
      await initDB();
      const db = getDB();
      const res = await db.query('SELECT * FROM job_applications ORDER BY applied_date DESC');
      setJobs(res.values as JobApplication[]);
      setLoading(false);
    }
    loadJobs();
  }, []);

  async function addJob(job: JobApplication) {
    ...
  }

  async function updateJob(job: JobApplication) {
    ...
  }

  async function deleteJob(id: number) {
    ...
  }

  return { jobs, loading, addJob, updateJob, deleteJob };
}
```

---

## 3. **AI Feature: Semantic Vector Search**

*Search like a human. Not like Ctrl+F.*

### Embedding Generator

```ts
import { Configuration, OpenAIApi } from 'openai';

const configuration = new Configuration({
  apiKey: 'YOUR_OPENAI_API_KEY',
});
const openai = new OpenAIApi(configuration);

export async function getEmbedding(text: string): Promise<number[]> {
  const response = await openai.createEmbedding({
    model: 'text-embedding-ada-002',
    input: text,
  });
  return response.data.data[0].embedding;
}
```

### Cosine Similarity

```ts
function cosineSimilarity(vecA: number[], vecB: number[]) {
  const dot = vecA.reduce((sum, a, i) => sum + a * vecB[i], 0);
  const magA = Math.sqrt(vecA.reduce((sum, a) => sum + a * a, 0));
  const magB = Math.sqrt(vecB.reduce((sum, b) => sum + b * b, 0));
  return dot / (magA * magB);
}
```

### Hook for Semantic Search

```ts
export function useSemanticSearch() {
  const [results, setResults] = useState<any[]>([]);

  async function search(text: string) {
    const queryEmbedding = await getEmbedding(text);
    const db = getDB();
    const res = await db.query('SELECT id, company_name, job_title, notes, embedding FROM job_applications');

    const scored = (res.values || []).map((job: any) => {
      if (!job.embedding) return { ...job, score: 0 };
      const embedding = JSON.parse(job.embedding);
      const score = cosineSimilarity(queryEmbedding, embedding);
      return { ...job, score };
    });

    scored.sort((a, b) => b.score - a.score);
    setResults(scored.slice(0, 5));
  }

  return { results, search };
}
```

---

## Final Thoughts

- **Local-first** means no servers, no latency, just speed!
- **Embeddings** turn notes into smart searchable content.
- **Hooks + SQLite** = React app with native-like experience.

Now go impress your lead — or at least make them say: *"Nice!"*

---

> *“Ship code like a pro, search jobs like a wizard.”*
