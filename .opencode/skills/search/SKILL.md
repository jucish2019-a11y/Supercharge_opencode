---
name: search
description: Implement full-text search — Algolia, Meilisearch, and database search with autocomplete, faceted filters, relevance tuning, and search UX patterns
---

## What I do

I implement search functionality in web applications:

- **Search engine selection** — Algolia vs Meilisearch vs database search — when to use each
- **Indexing** — What to index, when to index, sync strategies
- **Autocomplete** — Type-ahead search with instant results
- **Faceted filters** — Category, status, date range, multi-select filters
- **Relevance tuning** — Custom ranking, boosts, synonyms, typos
- **Search UX** — Search bar, results page, empty states, recent searches
- **Performance** — Debouncing, caching, pagination, highlighting

## When to use me

Use this skill when:
- Adding search to an application (projects, tasks, documents, users)
- Choosing between Algolia, Meilisearch, and database search
- Implementing autocomplete/type-ahead
- Building faceted filter UI (categories, status, date ranges)
- Tuning search relevance and ranking
- Designing the search experience (input, results, empty states)

## Search engine selection

| Engine | Best for | Latency | Cost | Setup |
|--------|----------|---------|------|-------|
| **Algolia** | Production apps, need instant search | <10ms | Free up to 10K records, then usage-based | SaaS, no infrastructure |
| **Meilisearch** | Self-hosted, cost-sensitive, medium scale | <50ms | Free (self-hosted) | Docker deploy, manage yourself |
| **PostgreSQL FTS** | Simple search, <10K records, no new infra | 10-100ms | Free (uses existing DB) | Built into PostgreSQL |
| **Typesense** | Self-hosted alternative to Algolia | <50ms | Free (self-hosted) | Docker deploy |

### Decision tree

```
Need instant autocomplete (< 50ms)?
├── Yes, at scale (>100K records)?
│   └── → Algolia (managed, fastest)
├── Yes, but want to self-host?
│   └── → Meilisearch or Typesense
└── No, simple search is fine?
    ├── < 10K records?
    │   └── → PostgreSQL full-text search
    └── > 10K records or need typo tolerance?
        └── → Meilisearch (free, self-hosted)
```

## Algolia integration

### Setup

```bash
npm install algoliasearch
```

```ts
// lib/algolia.ts
import algoliasearch from 'algoliasearch';

export const algolia = algoliasearch(
  process.env.NEXT_PUBLIC_ALGOLIA_APP_ID!,
  process.env.ALGOLIA_ADMIN_KEY!
);

export const searchClient = algoliasearch(
  process.env.NEXT_PUBLIC_ALGOLIA_APP_ID!,
  process.env.NEXT_PUBLIC_ALGOLIA_SEARCH_KEY! // Public, search-only key
);

export const projectIndex = algolia.initIndex('projects');
```

### Index configuration

```ts
// scripts/algolia-config.ts
await projectIndex.setSettings({
  // Searchable attributes — order matters (first = highest priority)
  searchableAttributes: [
    'name',
    'description',
    'tags',
    'ownerName',
  ],

  // Attributes for faceting (filtering)
  attributesForFaceting: [
    'status',
    'tags',
    'ownerId',
    'isPublic',
  ],

  // Custom ranking — applied when text relevance is equal
  customRanking: [
    'desc(updatedAt)',
    'desc(memberCount)',
  ],

  // Typo tolerance
  typoTolerance: true,
  minWordSizeforTypo: {
    minWordSizeFor1Typo: 4,
    minWordSizeFor2Typo: 8,
  },

  // Highlighting
  attributesToHighlight: ['name', 'description'],
  attributesToSnippet: ['description:50'],

  // Pagination
  paginationLimitedTo: 1000,
});
```

### Indexing records

```ts
// After creating/updating a project, sync to Algolia

async function indexProject(project: Project) {
  await projectIndex.saveObject({
    objectID: project.id, // Must match your DB ID
    name: project.name,
    description: project.description,
    tags: project.tags,
    status: project.status,
    ownerId: project.ownerId,
    ownerName: project.owner.name,
    isPublic: project.isPublic,
    memberCount: project.memberCount,
    updatedAt: Math.floor(project.updatedAt.getTime() / 1000),
  });
}

async function removeProjectFromIndex(projectId: string) {
  await projectIndex.deleteObject(projectId);
}

// Batch indexing (initial load or reindex)
async function reindexAllProjects() {
  const projects = await db.project.findMany({
    include: { owner: { select: { name: true } } },
  });

  const records = projects.map(project => ({
    objectID: project.id,
    name: project.name,
    description: project.description,
    tags: project.tags,
    status: project.status,
    ownerId: project.ownerId,
    ownerName: project.owner.name,
    isPublic: project.isPublic,
    memberCount: project.memberCount,
    updatedAt: Math.floor(project.updatedAt.getTime() / 1000),
  }));

  await projectIndex.saveObjects(records);
}
```

### Keep index in sync (after every mutation)

```ts
// After creating a project
async function createProject(data: CreateProjectInput) {
  const project = await db.project.create({ data, include: { owner: true } });
  await indexProject(project); // Sync to Algolia
  return project;
}

// After updating a project
async function updateProject(id: string, data: UpdateProjectInput) {
  const project = await db.project.update({ where: { id }, data, include: { owner: true } });
  await indexProject(project); // Sync to Algolia
  return project;
}

// After deleting a project
async function deleteProject(id: string) {
  await db.project.delete({ where: { id } });
  await removeProjectFromIndex(id); // Remove from Algolia
}
```

### Search UI component

```tsx
import { useInstantSearch, Configure, Hits, RefinementList, Pagination, Highlight } from 'react-instantsearch';

function SearchPage() {
  return (
    <InstantSearch searchClient={searchClient} indexName="projects">
      <Configure hitsPerPage={20} />

      <div className="flex gap-6">
        {/* Filters sidebar */}
        <aside className="w-64 shrink-0">
          <h3>Status</h3>
          <RefinementList attribute="status" />

          <h3>Tags</h3>
          <RefinementList attribute="tags" limit={10} />
        </aside>

        {/* Results */}
        <main className="flex-1">
          <SearchBox />
          <Hits hitComponent={ProjectHit} />
          <Pagination />
        </main>
      </div>
    </InstantSearch>
  );
}

function SearchBox() {
  const { query, refine } = useSearchBox();

  return (
    <div className="relative">
      <SearchIcon className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
      <input
        type="search"
        value={query}
        onChange={(e) => refine(e.target.value)}
        placeholder="Search projects..."
        className="w-full pl-10 pr-4 py-2 border rounded-lg"
      />
    </div>
  );
}

function ProjectHit({ hit }: { hit: any }) {
  return (
    <a href={`/projects/${hit.objectID}`} className="block p-4 hover:bg-muted rounded-lg">
      <h3>
        <Highlight attribute="name" hit={hit} />
      </h3>
      <p className="text-sm text-muted-foreground">
        <Highlight attribute="description" hit={hit} />
      </p>
      <div className="flex gap-2 mt-2">
        {hit.tags.map((tag: string) => (
          <span key={tag} className="text-xs px-2 py-0.5 bg-secondary rounded">{tag}</span>
        ))}
      </div>
    </a>
  );
}
```

## Meilisearch integration (self-hosted)

### Setup

```bash
# Docker
docker run -d \
  -p 7700:7700 \
  -e MEILI_MASTER_KEY=your-master-key \
  -v meili_data:/meili_data \
  getmeili/meilisearch:v1.5
```

```ts
// lib/meilisearch.ts
import { MeiliSearch } from 'meilisearch';

export const meili = new MeiliSearch({
  host: process.env.MEILISEARCH_URL || 'http://localhost:7700',
  apiKey: process.env.MEILISEARCH_MASTER_KEY!,
});

export const projectIndex = meili.index('projects');
```

### Index configuration

```ts
await projectIndex.updateSettings({
  searchableAttributes: ['name', 'description', 'tags', 'ownerName'],
  filterableAttributes: ['status', 'tags', 'ownerId', 'isPublic'],
  sortableAttributes: ['updatedAt', 'memberCount'],
  rankingRules: [
    'words',
    'typo',
    'proximity',
    'attribute',
    'sort',
    'exactness',
    'updatedAt:desc',
  ],
  pagination: { maxTotalHits: 1000 },
});
```

### Search

```ts
const results = await projectIndex.search('project name', {
  filter: ['status = active', 'isPublic = true'],
  sort: ['updatedAt:desc'],
  limit: 20,
  attributesToHighlight: ['name', 'description'],
});
```

## PostgreSQL full-text search (simple, no external service)

### Setup

```sql
-- Add search vector column
ALTER TABLE projects ADD COLUMN search_vector tsvector;

-- Create GIN index for fast search
CREATE INDEX idx_projects_search ON projects USING GIN(search_vector);

-- Create trigger to auto-update search vector
CREATE OR REPLACE FUNCTION projects_search_vector_update() RETURNS trigger AS $$
BEGIN
  NEW.search_vector :=
    setweight(to_tsvector('english', COALESCE(NEW.name, '')), 'A') ||
    setweight(to_tsvector('english', COALESCE(NEW.description, '')), 'B') ||
    setweight(to_tsvector('english', COALESCE(NEW.tags_text, '')), 'C');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER projects_search_vector_trigger
  BEFORE INSERT OR UPDATE OF name, description, tags
  ON projects
  FOR EACH ROW
  EXECUTE FUNCTION projects_search_vector_update();
```

### Search with Prisma raw query

```ts
async function searchProjects(query: string, userId: string, limit = 20) {
  const results = await db.$queryRaw`
    SELECT p.*, ts_rank(p.search_vector, plainto_tsquery('english', ${query})) AS rank
    FROM projects p
    INNER JOIN project_members pm ON p.id = pm.project_id
    WHERE pm.user_id = ${userId}
      AND p.search_vector @@ plainto_tsquery('english', ${query})
    ORDER BY rank DESC
    LIMIT ${limit}
  `;

  return results;
}
```

### Limitations of PostgreSQL FTS

```
✓ Good for: < 10K records, simple search, no new infrastructure needed
✓ Good for: Exact and stemmed word matching
✗ Poor for: Typo tolerance (no built-in fuzzy matching)
✗ Poor for: Faceted search (need to build filter UI manually)
✗ Poor for: Relevance tuning (limited ranking controls)
✗ Poor for: Instant autocomplete (< 50ms at scale)
✗ Poor for: Highlighting (need to implement manually)

For any of these "poor" items, use Algolia or Meilisearch.
```

## Autocomplete / type-ahead

```tsx
function useSearch(query: string, indexName: string) {
  const [results, setResults] = useState<any[]>([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (query.length < 2) {
      setResults([]);
      return;
    }

    const timer = setTimeout(async () => {
      setLoading(true);
      try {
        const { hits } = await searchClient.search([
          { indexName, query, params: { hitsPerPage: 5 } },
        ]);
        setResults(hits[0].hits);
      } finally {
        setLoading(false);
      }
    }, 150); // 150ms debounce

    return () => clearTimeout(timer);
  }, [query, indexName]);

  return { results, loading };
}

function SearchAutocomplete() {
  const [query, setQuery] = useState('');
  const [isOpen, setIsOpen] = useState(false);
  const { results, loading } = useSearch(query, 'projects');

  return (
    <div className="relative">
      <input
        type="search"
        value={query}
        onChange={(e) => { setQuery(e.target.value); setIsOpen(true); }}
        onFocus={() => setIsOpen(true)}
        onBlur={() => setTimeout(() => setIsOpen(false), 200)}
        placeholder="Search projects..."
        className="w-full px-4 py-2 border rounded-lg"
      />
      {isOpen && query.length >= 2 && (
        <div className="absolute top-full mt-1 w-full bg-white border rounded-lg shadow-lg z-50">
          {loading ? (
            <div className="p-4 text-sm text-muted-foreground">Searching...</div>
          ) : results.length === 0 ? (
            <div className="p-4 text-sm text-muted-foreground">No results found</div>
          ) : (
            <ul>
              {results.map((hit: any) => (
                <li key={hit.objectID}>
                  <a href={`/projects/${hit.objectID}`} className="block px-4 py-2 hover:bg-muted">
                    <Highlight attribute="name" hit={hit} />
                    <span className="text-xs text-muted-foreground ml-2">{hit.status}</span>
                  </a>
                </li>
              ))}
            </ul>
          )}
        </div>
      )}
    </div>
  );
}
```

## Search UX patterns

### Recent searches

```ts
function useRecentSearches(maxItems = 5) {
  const [recent, setRecent] = usePersistedState<string[]>('recent-searches', []);

  const addSearch = (query: string) => {
    setRecent(prev => {
      const next = [query, ...prev.filter(s => s !== query)].slice(0, maxItems)];
      return next;
    });
  };

  const clearRecent = () => setRecent([]);

  return { recent, addSearch, clearRecent };
}

// Search bar shows recent searches when focused with no query
// Clicking a recent search fills the query and searches
// Clear button removes all recent searches
```

### Empty states

```
No query entered:
  → Show recent searches + popular/trending items

No results found:
  → "No results for 'xyz'"
  → Suggestions: "Check your spelling", "Try more general terms"
  → Show popular items as fallback

Loading:
  → Skeleton cards matching result shape

Error:
  → "Search is temporarily unavailable"
  → Fallback: client-side filter of cached data
```

### Keyboard shortcuts

```
Cmd+K / Ctrl+K: Focus search bar (global)
Escape: Close search, clear query
Arrow up/down: Navigate results
Enter: Select result
```

## Relevance tuning

### Algolia custom ranking

```ts
await projectIndex.setSettings({
  customRanking: [
    'desc(memberCount)',      // More members = higher rank
    'desc(updatedAt)',         // Recently updated = higher rank
    'desc(isPublic)',          // Public projects rank higher
  ],
});
```

### Boosting

```ts
// Boost projects the user is a member of
const results = await projectIndex.search(query, {
  optionalFilters: [`ownerId:${userId}`],  // Boost user's own projects
});
```

### Synonyms

```ts
await projectIndex.setSettings({
  synonyms: [
    ['bug', 'defect', 'issue'],
    ['task', 'todo', 'item'],
    ['doc', 'document', 'documentation'],
  ],
});
```

### One-way synonyms (query expansion)

```ts
await projectIndex.saveSynonyms([
  {
    objectID: 'repo-synonym',
    type: 'oneWaySynonym',
    input: 'repo',
    synonyms: ['repository', 'project'],
  },
]);
```

## Quality checklist

- [ ] Search works with typos (fuzzy matching)
- [ ] Autocomplete with 150ms debounce
- [ ] Search results highlighted (matched terms visible)
- [ ] Faceted filters work (status, tags, date range)
- [ ] Recent searches saved and displayed
- [ ] Empty state shows suggestions
- [ ] Keyboard navigation (Cmd+K, arrows, enter)
- [ ] Index updated on every create/update/delete
- [ ] Index rebuild script for full reindex
- [ ] Search analytics tracked (popular searches, zero-result queries)
- [ ] Mobile search works (full-screen search on mobile)
- [ ] Pagination or infinite scroll for results
- [ ] Search bar accessible (label, ARIA)
- [ ] Rate limiting on search API (prevent abuse)

## Anti-patterns I avoid

- Using LIKE '%query%' for search — no ranking, no typo tolerance, slow
- No debounce on autocomplete — fires on every keystroke
- Searching the database directly for > 10K records — use a search engine
- Not highlighting matched terms — users can't see why a result matched
- Showing "no results" without suggestions — help the user refine
- Not syncing the index on mutations — stale results
- Not handling zero-result queries — track them and add synonyms
- Loading all results at once — paginate (20 results per page)
- Adding search after the product is built — design search into the data model from the start
- Using Algolia admin key on the client — always use the search-only key