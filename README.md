# lists

List utilities for SSPUR that the builtin `List` methods don't cover: splitting and sliding windows, run grouping, counting into maps, zips of every shape, order-preserving set and multiset operations, combinatorics with a safety limit, binary search by key, and index edits that never trap. It is the SSPUR counterpart of Guava's `Lists`/`Iterables`, Python's `itertools` and `more-itertools`, and lodash's array functions.

Every function is pure and generic. A function that takes a lambda performs whatever the lambda performs (`f: A -> B ! e`), so you can pass logging or failing functions. Out-of-range indexes and counts never trap: each function says what it does with them (usually clamping, or returning the list unchanged). The combinatoric functions have a `pre` clause that limits their output to `combinatorics_limit()` (1,000,000) lists.

## Example

```sspur
use lists.{chunk_by, top_k, sliding, combinations, index_by, split_on}

type Order = {id: Int, customer: Str, cents: Int} derive Eq, Show

fn main() -> Unit ! log
= do
  orders = [Order{id: 1, customer: "ana", cents: 1200}, Order{id: 2, customer: "ana", cents: 300}, Order{id: 3, customer: "bo", cents: 4500}, Order{id: 4, customer: "cy", cents: 800}]
  log(top_k(orders, 2, _.cents).map(_.id).str)
  log(chunk_by(orders, _.customer).map(g => "{g.0}: {g.1.len}").join(", "))
  log(index_by(orders, _.id).get(3).map(_.customer).str)
  log(sliding([1, 2, 3, 4, 5], 2, 2, true).str)
  log(combinations(["a", "b", "c"], 2).str)
  log(split_on(["x", "", "y", "z", ""], "").str)
  log(lists.cartesian([1, 2], ["s", "m"]).str)
```

Output (the same natively and with `--interp`):

```
[3, 1]
ana: 2, bo: 1, cy: 1
some("bo")
[[1, 2], [3, 4], [5]]
[["a", "b"], ["a", "c"], ["b", "c"]]
[["x"], ["y", "z"], []]
[(1, "s"), (1, "m"), (2, "s"), (2, "m")]
```

## API

Each function has a doc comment with its exact edge-case behaviour; `sspur api` lists them.

### Splitting and windows

| Function | What it does |
|---|---|
| `split_at(xs, i)` | `(take(i), drop(i))`, `i` clamped |
| `split_while(xs, p)`, `split_until(xs, p)` | Haskell's `span` and `break` |
| `split_when(xs, p)` | split between neighbours where `p(a, b)` holds |
| `split_on(xs, sep)`, `split_where(xs, p)` | split at separators, dropping them, like `Str.split` |
| `split_parts(xs, n)` | exactly `n` parts of nearly equal length |
| `sliding(xs, size, step, partial)` | windows with a step, optionally keeping shortened tail windows |
| `chunks_padded(xs, n, fill)` | `chunks` with the last chunk padded |
| `chunk_by(xs, key)` | runs of equal keys, with the key |
| `rle(xs)`, `rle_decode(runs)` | run-length encoding |
| `tails(xs)`, `inits(xs)` | all suffixes, all prefixes |
| `take_last`, `drop_last`, `take_last_while`, `drop_last_while` | the end of a list |
| `pad_to(xs, n, fill)`, `pad_front_to(xs, n, fill)` | pad to a minimum length |

### Duplicates

| Function | What it does |
|---|---|
| `dedup(xs)`, `dedup_by(xs, key)` | collapse runs of equal neighbours |
| `distinct_by(xs, key)` | first element per key |
| `duplicates(xs)` | values that occur more than once |
| `all_equal(xs)`, `all_distinct(xs)` | uniformity checks |
| `single(xs)` | the only element, if there is exactly one |

### Counting and grouping

| Function | What it does |
|---|---|
| `frequencies(xs)` | `Map` of value to count |
| `count_by(xs, key)`, `count_where(xs, p)` | counts by key, or matching a predicate |
| `index_by(xs, key)` | `Map` of key to element, the last wins |
| `associate(xs, f)` | `Map` from the pairs `f` gives, the last wins |
| `group_into(xs, key)` | `Map` of key to its elements in input order |

### Aggregates

| Function | What it does |
|---|---|
| `sum_by(xs, f)`, `sum_by_f64(xs, f)` | sums, the float one compensated (Kahan-Babuska) |
| `min_by`, `max_by`, `min_max`, `min_max_by` | extremes, the first among equals |
| `top_k(xs, k, key)`, `bottom_k(xs, k, key)` | the `k` best by key, stable |
| `largest(xs, k)`, `smallest(xs, k)` | the `k` largest or smallest elements |

### Combining

| Function | What it does |
|---|---|
| `interleave(xs, ys)`, `interleave_all(xss)` | alternate elements, round-robin over many lists |
| `intersperse(xs, sep)`, `intercalate(xss, sep)` | separators between elements or lists |
| `zip3`, `zip_with`, `zip_longest`, `zip_longest_opt` | zips of three lists, with a function, or to the longer length |
| `unzip(ps)`, `unzip3(ts)` | split pairs and triples |
| `transpose(rows)` | rows to columns, ragged rows allowed |
| `pairwise(xs)` | adjacent pairs |

### Combinatorics

| Function | What it does |
|---|---|
| `cartesian(xs, ys)`, `cartesian_all(xss)` | cartesian products |
| `permutations(xs, k)`, `combinations(xs, k)`, `combinations_with_replacement(xs, k)` | in lexicographic order of positions, as in `itertools` |
| `power_set(xs)` | every sublist, by size; at most 20 elements |
| `count_permutations(n, k)`, `count_combinations(n, k)` | exact counts, `none` when they don't fit an `Int` |
| `combinatorics_limit()` | the output limit, 1,000,000 |

### Building and mapping

| Function | What it does |
|---|---|
| `replicate(x, n)`, `cycle(xs, n)`, `cycle_take(xs, n)` | repetition |
| `map_indexed(xs, f)`, `filter_indexed(xs, p)` | with the index |
| `filter_map(xs, f)`, `compact(xs)`, `find_map(xs, f)` | working with `Opt` |
| `partition_map(xs, f)`, `partition_results(rs)` | split `ok`s from `err`s |
| `sequence_opt(xs)`, `sequence_res(rs)` | all or nothing |

### Index edits (never trap)

| Function | What it does |
|---|---|
| `at(xs, i)` | element with negative indexes from the end |
| `insert_at`, `insert_all_at` | insert, the index clamped |
| `remove_at`, `remove_first` | remove by index or the first equal element |
| `set_at`, `update_at` | replace one element |
| `swap(xs, i, j)`, `move_item(xs, from, to)` | reorder |

### Searching

| Function | What it does |
|---|---|
| `find_last`, `find_last_index`, `last_index_of` | search from the end |
| `indices_where(xs, p)`, `indices_of(xs, x)` | every matching index |
| `partition_point(xs, p)` | binary search for the end of the `p` prefix |
| `lower_bound_by`, `upper_bound_by`, `binary_search_by` | binary search by key |
| `insert_sorted(xs, x)`, `insert_sorted_by(xs, x, key)` | stable sorted insertion |
| `index_of_sublist`, `last_index_of_sublist`, `contains_sublist` | contiguous sublists |
| `has_prefix`, `has_suffix` | `starts_with` and `ends_with` for lists |
| `count_sublist`, `replace_sublist` | non-overlapping occurrences, left to right |

### Folds and order

| Function | What it does |
|---|---|
| `fold_right`, `reduce`, `reduce_right`, `scan_right` | right folds and folds without a start value |
| `is_sorted`, `is_strictly_sorted`, `is_sorted_by`, `is_sorted_with` | sortedness by `Ord`, key or comparator |
| `argsort`, `argsort_by`, `ranks`, `dense_ranks` | sorting permutations and ranks |
| `compare_lists(xs, ys)` | lexicographic comparison |
| `merge_by(xs, ys, key)`, `merge_all(xss)` | stable merges of sorted lists |

Functions with an `Ord` bound use the type's `Ord` impl, so a hand-written order (a descending wrapper, say) is respected.

### Set and multiset operations

| Function | What it does |
|---|---|
| `union`, `intersect`, `difference`, `sym_difference` | distinct results in first-seen order |
| `contains_all`, `contains_any`, `is_disjoint` | membership |
| `multi_intersect`, `multi_difference`, `multi_union` | keeping duplicates by count, in input order |
| `is_permutation(xs, ys)` | the same elements with the same counts |

### Random

| Function | What it does |
|---|---|
| `sample(xs, k, seed)`, `sample_ordered(xs, k, seed)` | `k` distinct positions, pure in the seed like the builtin `shuffle` |

## Out of scope

- Lazy and infinite sequences: the standard library's `View` (`xs.view`, `iota`, `iterate`, `take_while`, `chain`) and generators (`yield`) cover them, so everything here is eager.
- What the builtins already do: `chunks`, `windows`, `partition`, `scan`, `flatten`, `take_while`, `drop_while`, `rotate`, `binary_search`, `lower_bound`, `upper_bound`, `group_by`, `counts`, `merge`, `next_perm`, `unique`, `shuffle`, `choice`, `slice`, `find_index`, `index_of`, `zip`, `enumerate`.
- Statistics (mean, median, percentiles) belong to a numbers package; maps of maps to a maps package.
- Mutable, in-place algorithms (`nth_element`, in-place sorts): SSPUR lists are immutable values.
- Weighted sampling and random distributions: use the builtin `rand_weighted` and `Rng`.

Performance notes: index-based results are built with `map`, `filter` and `flat_map`, so they are linear in both tiers. `index_of_sublist` and friends are naive (O(n * m) in the worst case). In the interpreter, `tails` and `inits` copy each suffix and prefix.

## Install

```
sspur add lists
```

MIT licensed.
