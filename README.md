<img src="assets/logo.png" style="height:75px" />

# RipBtree — Blazing Fast, In‑Memory B+tree Library for TypeScript/JavaScript

### Why RipBtree

- High‑performance ordered map in TypeScript/JavaScript with B+Tree data structure
- Efficient range scans and `$ORDER`‑style iteration (`next`/`prev`, lower/upper‑bound)
- Mutation speed suitable for heavy update workloads (SET/KILL‑like patterns)
- Optional cheap snapshots (via clone) to support overlay/frame semantics
- Small, dependency‑light library with clear API and strong TypeScript types

### Core Concepts

- **B+Tree map**: Keys are ordered and stored in nodes (arrays) for cache‑friendly performance
- **Comparators**: Plug‑in comparator (e.g., bytewise memcmp for `Uint8Array` keys)
- **Iterators**: Seek by key and iterate forwards or backwards in key order
- **Snapshots (clone)**: Create an immutable view for overlays; restore in O(1)

### Highlights

- **Performance**
  - O(log n) set/get/delete; forward iteration O(1) amortized per step
  - Array‑packed nodes minimize per‑entry overhead
  - Fast range queries and ORDER‑like traversal
- **Simplicity**
  - Pure TypeScript/ESM, no native dependencies
  - Clear, well‑documented API; strong typings
- **Flexibility**
  - Custom comparators (e.g., bytewise for binary keys)
  - Clone for snapshot/overlay use cases

### Quickstart

Install (as a library):

```bash
npm install ripbtree
```

Minimal example (TypeScript):

```ts
import { BTree } from "ripbtree";

// Bytewise comparator for Uint8Array keys
const cmp = (a: Uint8Array, b: Uint8Array) => {
  const n = Math.min(a.length, b.length);
  for (let i = 0; i < n; i++) { const d = a[i]! - b[i]!; if (d) return d; }
  return a.length - b.length;
};

const tree = new BTree<Uint8Array, Uint8Array>(cmp);

const enc = new TextEncoder();
tree.set(enc.encode("A"), enc.encode("1"));
tree.set(enc.encode("B"), enc.encode("2"));

// ORDER‑style iteration
for (const [k, v] of tree.entriesFrom(enc.encode("A"))) {
  console.log(k, v);
}

// Snapshot / overlay
const snap = tree.clone(); // immutable view
const restored = snap;     // O(1) restore semantics via clone handles/patterns
```

### API (overview)

Core methods (subset; see docs):
- `constructor(comparator)`
- `get(key)` / `set(key, value)` / `delete(key)`
- `has(key)` / `size`
- `entries()` / `entriesFrom(key)` / `entriesReversed()`
- `nextHigherKey(key)` / `nextLowerKey(key)`
- `lowerBound(key)` / `upperBound(key)`
- `range(lowInclusive?, highExclusive?)`
- `clone()`

Additional APIs
- `setPairs(pairs: Iterable<[K,V]>)`: bulk insert/update
- `deleteRange(lowInclusive, highExclusive)`: remove a key range efficiently
- `getPairOrNextLower(key)` / `getPairOrNextHigher(key)`: floor/ceil lookups
- `nextHigherPair(key)` / `nextLowerPair(key)`: step to next pair in either direction
- `diffAgainst(other, onlyThis?, onlyOther?, different?) → R|undefined`:
  - Walks both trees in order, skipping shared subtrees
  - Invokes callbacks for keys only in this/other, or keys with different values
  - Supports early exit: return `{ break: R }` from a callback to stop and return R

### Best practices & notes

- Use a comparator matching your key format (e.g., bytewise for binary keys)
- For snapshots/overlays, prefer `clone()` before mutating; restore by keeping the prior clone
- For large batch inserts, consider inserting keys in sorted order for fewer node splits

Comparators
- Default comparator supports JS primitives and dates; for binary keys (`Uint8Array`) use bytewise lexicographic:

```ts
const bytewise = (a: Uint8Array, b: Uint8Array) => {
  const n = Math.min(a.length, b.length);
  for (let i = 0; i < n; i++) { const d = a[i]! - b[i]!; if (d) return d; }
  return a.length - b.length;
};
```

Snapshots & cloning
- `clone()` marks nodes shared (copy‑on‑write on mutation); cheap, ideal for overlays
- `greedyClone(force?)` proactively duplicates nodes to avoid sharing

Tuning
- `maxNodeSize` (fanout) controls node capacity; higher fanout reduces height (fewer pointer hops) at the cost of larger node copies on split/merge. Defaults are reasonable; tune after profiling.

Performance tips
- Prefer sorted bulk inserts for large loads (`setPairs`)
- Reuse arrays returned from pair APIs where exposed to minimize GC churn
- Keep comparator pure and fast; avoid allocations in hot paths

### Build & tests

- Pure TypeScript; build via your project’s TS/Bun/Node toolchain
- Unit tests can be adapted from upstream B+Tree libraries or your usage scenarios

### License & attribution

- Licensed under the **MIT License** (see `LICENSE`).
- RipBtree is based on high‑quality B‑tree concepts and draws inspiration from the `sorted-btree` project.

### Project layout

- `src/btree.ts` — core B‑tree map implementation
- `src/sorted-array.ts` — array helpers used by the B‑tree
- `src/interfaces.d.ts` — type declarations consumed by the tree
- `package.json`, `tsconfig.json` — build metadata
- `LICENSE` — license text

### Support

Issues, questions, or performance tuning help? Open a discussion or issue on projects using RipBtree and reference this repository.
