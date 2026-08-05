# 🧵 String Algorithms — Master Tracker (TLA Eliminator 7.0)

> **Philosophy**: You are NOT memorizing 21 CSES solutions. You are mastering **9 core ideas**.
> Every one of the 21 problems below is just one of those 9 ideas wearing a costume.
> Once you see through the costume, CSES, Codeforces, and LeetCode all become the same game.

> **How this file works**: This is your map + checklist. The actual *teaching* of each core
> idea (the "aha" moment) happens interactively in chat, one pattern at a time — in the exact
> order listed below. Don't skip ahead. Come back to this file to log progress and pick your
> next practice problem.

---

## 🗺️ Part 0 — Why This Order (Not the CSES Order)

CSES lists problems in this order:
`Word Combinations → String Matching → Borders → Periods → Minimal Rotation → Palindromes → Required Substring → Palindrome Queries → Aho-Corasick trio → Suffix Array cluster → String Transform`

That order is fine for *solving CSES top-to-bottom*, but it's a bad order for *learning*, because
it jumps between unrelated ideas (rotations, then palindromes, then automaton counting, then
hashing+segtree, then back to suffix structures). We will instead go in **dependency order** —
each pattern below builds on the intuition of the one before it:

```
1. String Hashing        → the universal tool. Everything else either replaces it or builds on it.
2. Prefix Function (KMP) → "how much does this string overlap with itself?"
3. Z-Function            → same question as KMP, different lens (reinforces #2)
4. Trie                  → organizing MANY strings instead of one
5. Aho-Corasick          → Trie + KMP's "failure" idea, generalized to many patterns
6. Manacher's Algorithm  → palindromes, standalone, linear time
7. Hashing + Segment Tree→ revisit #1, now with UPDATES
8. Suffix Array + LCP    → the "give me the truth about ALL substrings at once" tool
9. Booth's Algorithm/BWT → comparing rotations (closely tied to #8's mindset)
10. [Bonus] Counting DP  → Distinct Subsequences — not a string-structure pattern, pure DP
```

---

## 📊 Part 1 — The Master Progress Tracker

| # | Pattern | Core Question It Answers | CSES Problems | Interview Relevance (Google/Amazon/MS/Adobe) | Status |
|---|---------|---------------------------|----------------|-----------------------------------------------|--------|
| 1 | String Hashing | "Are these two substrings equal, fast?" | (tool only, no dedicated CSES problem) | 🟢 High | ⬜ |
| 2 | Prefix Function (KMP) | "Where does this string overlap with itself?" | String Matching, Finding Borders, Finding Periods, Required Substring | 🟡 Medium-High | ⬜ |
| 3 | Z-Function | Same as KMP, different construction | String Matching (redo) | 🟡 Medium | ⬜ |
| 4 | Trie | "How do I organize/search many strings by prefix?" | Word Combinations | 🟢 Very High | ⬜ |
| 5 | Aho-Corasick | "Where do MANY patterns occur in one text?" | Finding Patterns, Counting Patterns, Pattern Positions | 🟠 Low-Medium (rare direct ask) | ⬜ |
| 6 | Manacher's Algorithm | "What's the longest palindrome, everywhere, in O(n)?" | Longest Palindrome, All Palindromes | 🟠 Low (O(n²) usually accepted) | ⬜ |
| 7 | Hashing + Segment Tree | "Palindrome/equality checks, but the string changes" | Palindrome Queries | 🟠 Low-Medium | ⬜ |
| 8 | Suffix Array + LCP (Kasai's) | "Tell me everything about all substrings/suffixes at once" | Distinct Substrings, Repeating Substring, String Functions, Inverse Suffix Array, Substring Order I, Substring Order II, Substring Distribution | 🟠 Low (too long for 45-min interview, but 1 famous LC problem uses it) | ⬜ |
| 9 | Booth's Algorithm / Rotations (BWT) | "What's the canonical form among all rotations?" | Minimal Rotation, String Transform | 🔴 Very Low direct ask | ⬜ |
| 10 | [Bonus] Counting DP | "How many distinct subsequences exist?" | Distinct Subsequences | 🟡 Medium (classic DP, asked as-is) | ⬜ |

> **Read the relevance column honestly**: Patterns 1–4 are your highest-leverage investment for
> Google/Amazon/Adobe interviews. Patterns 5–9 are competitive-programming depth — they make you
> a *much* stronger problem solver and will make every LeetCode string problem trivial, but don't
> expect Aho-Corasick or Suffix Arrays to be asked cold in a 45-minute SDE-2 loop. You're building
> them because deep understanding of 5–9 is exactly what makes 1–4 feel effortless.

---

## 🧭 Part 2 — The Constraint → Pattern Decision Chart

> This is the chart you asked for. Read the constraints/keywords in a problem, find the row, that's your tool.

| Signal in the Problem | Typical Constraints | → Use This Pattern |
|---|---|---|
| "Does pattern P occur in text T? How many times? Where?" (ONE pattern) | `\|T\|, \|P\| ≤ 10^6` | KMP or Z-function |
| Same, but MANY patterns at once | `Σ\|patterns\| ≤ 10^5–10^6` | Aho-Corasick |
| "Are substring A and substring B equal?" asked many times | `n, q ≤ 10^5–10^6` | String Hashing |
| Above, but the string also gets **updated** | `n, q ≤ 10^5–10^6`, point updates | Hashing + Segment Tree |
| "Longest palindrome" / "is this substring a palindrome" (static string) | `n ≤ 10^6` | Manacher's Algorithm |
| "Find/organize/search many words by prefix" | `Σ\|words\| ≤ 10^6` | Trie |
| "Count/find/rank ALL distinct substrings", "kth substring", "LCP of suffixes" | `n ≤ 10^5` | Suffix Array + LCP (Kasai's) |
| "Lexicographically smallest rotation", "cyclic string canonical form" | `n ≤ 10^6` | Booth's Algorithm |
| "Self-overlap", "borders", "periods", "shortest string that repeats to form this" | `n ≤ 10^6` | Prefix Function (KMP) |

> **The one meta-skill**: 90% of "which string pattern?" confusion disappears once you ask:
> *"Am I comparing the string to itself, comparing it to another string, comparing it to MANY
> other strings, or reasoning about ALL of its substrings simultaneously?"* Those four questions
> map almost 1:1 to KMP, Hashing, Aho-Corasick, and Suffix Arrays respectively.

---
---

## 🔷 PATTERN 1 — String Hashing (Rolling / Polynomial Hash)

### 🎯 Core Idea (preview — full teaching in chat)
Turn a string into a number, such that comparing two substrings for equality becomes comparing
two numbers in O(1) instead of O(length). It's a **lossy compression that's safe enough,
probabilistically, for competitive programming.**

### 💼 Product-Company Relevance
Directly powers: *Repeated DNA Sequences*, *Longest Duplicate Substring*, *Distinct Echo
Substrings*, *Find Substring With Given Hash Value* — all real interview-style problems where
brute-force substring comparison is too slow.

### 🧪 Codeforces Practice Set
- [ ] CF 1200E — Compress Words (1600) — merge strings using hash-based overlap detection
- [ ] CF 514C — Watto and Mechanism (1900) — Trie+hash, strings differing by exactly one char
- [ ] CF 25E — Test (2100) — combine 3 patterns using hashing/KMP ideas

> ⚠️ Honest note: Codeforces doesn't tag "hashing" problems as generously as CSES does — most
> hashing problems on CF are bundled with `strings` or `string suffix structures`. Once you finish
> these 3, self-source more via **Codeforces → Problemset → filter tag `hashing`, rating 1400–1900**.

---

## 🔷 PATTERN 2 — Prefix Function (KMP Algorithm)

### 🎯 Core Idea (preview)
For every prefix of the string, answer: *"What's the longest proper prefix of this string that is
also a suffix of it?"* That single array unlocks pattern matching, borders, and periods — they're
all the same array read differently.

### 🧩 CSES Problems
- [ ] String Matching
- [ ] Finding Borders
- [ ] Finding Periods
- [ ] Required Substring *(KMP automaton + counting DP — the hardest of the four)*

### 💼 Product-Company Relevance
Directly powers: *Implement strStr()*, *Repeated Substring Pattern*, *Shortest Palindrome*,
*Longest Happy Prefix*. This is genuinely one of the highest-ROI patterns for interviews.

### 🧪 Codeforces Practice Set
- [ ] CF 471D — MUH and Cube Walls (1400) — easiest real KMP application, start here
- [ ] CF 126B — Password (1700) — borders that also appear as a substring in the middle
- [ ] CF 1200E — Compress Words (1600) — re-solve using KMP instead of hashing
- [ ] CF 616E — Om Nom and Necklace (2200) — periods via the failure function
- [ ] CF 432D — Prefixes and Suffixes (2000) — count occurrences of every border
- [ ] CF 535D — Tavas and Malekas (2200) — KMP + combinatorics, harder finisher

---

## 🔷 PATTERN 3 — Z-Function

### 🎯 Core Idea (preview)
Same question as KMP asked from a different angle: *"Starting at position i, how long is the
match with the string's own prefix?"* Whatever KMP can do, Z can usually do too — learning both
is about building flexibility, not learning something new.

### 🧩 CSES Problems
- [ ] String Matching *(redo this one using Z-function instead of KMP)*

### 🧪 Codeforces Practice Set — Deliberate Practice, Not New Problems
> The best way to learn Z-function is NOT new problems — it's re-solving 3 problems from your
> KMP list using Z instead. This forces you to feel *why* you'd reach for one over the other.
- [ ] Re-solve CF 126B — Password, using Z-function
- [ ] Re-solve CF 1200E — Compress Words, using Z-function
- [ ] Re-solve CF 535D — Tavas and Malekas, using Z-function

---

## 🔷 PATTERN 4 — Trie (Prefix Tree)

### 🎯 Core Idea (preview)
Store many strings as a tree of characters so that shared prefixes are shared nodes. Turns
"does any word in my dictionary start with X?" into a simple tree walk.

### 🧩 CSES Problems
- [ ] Word Combinations *(Trie/hashing + DP — "word break," counted)*

### 💼 Product-Company Relevance
Directly powers: *Implement Trie (Prefix Tree)*, *Word Search II*, *Design Add and Search Words
Data Structure*, *Replace Words*, *Stream of Characters*, autocomplete-style system design
questions. **This is one of the single highest-value patterns for Google/Amazon interviews —
more common on LeetCode than on Codeforces.**

### 🧪 Codeforces Practice Set
- [ ] CF 706D — Vasiliy's Multiset (2100) — binary trie over XOR (numbers, not words — same DS)
- [ ] CF 665E — Beautiful Subarrays (1900) — binary trie + prefix XOR

> ⚠️ Honest note: Codeforces rarely poses "pure dictionary Trie" problems — CP treats Trie as too
> basic to be its own problem, so it usually shows up disguised as a **binary trie over XOR**
> (as above) or bundled inside Aho-Corasick. Your deepest Trie practice for the *word-dictionary*
> flavor will come from CSES Word Combinations now, and LeetCode later.

---

## 🔷 PATTERN 5 — Aho-Corasick Automaton

### 🎯 Core Idea (preview)
Build ONE Trie for ALL your patterns, then add KMP-style "failure links" between nodes so that
after a mismatch, you don't restart from scratch — you jump to the next-best matching pattern.
It's KMP generalized from 1 pattern to N patterns.

### 🧩 CSES Problems
- [ ] Finding Patterns
- [ ] Counting Patterns
- [ ] Pattern Positions

### 💼 Product-Company Relevance
Rarely asked to *build* in an interview (too much to implement in 45 minutes), but the underlying
idea ("match many patterns in one pass") is the conceptual basis for *Stream of Characters* and
appears in "how would you scale this to 10,000 patterns?" follow-up questions.

### 🧪 Codeforces Practice Set
- [ ] CF 1202E — You Are Given Some Strings... (2400)
- [ ] CF 963D — Frequency of String (2900 — hard, treat as a stretch goal, not a requirement)

---

## 🔷 PATTERN 6 — Manacher's Algorithm

### 🎯 Core Idea (preview)
Palindromes centered at adjacent positions overlap and share information. Manacher exploits that
overlap (very similar spirit to how KMP reuses previously computed matches) to find the radius of
every palindrome in the string in O(n), instead of O(n²).

### 🧩 CSES Problems
- [ ] Longest Palindrome
- [ ] All Palindromes

### 💼 Product-Company Relevance
*Longest Palindromic Substring* and *Palindromic Substrings* are common LeetCode/interview asks
— but the O(n²) "expand around center" approach is usually accepted. Knowing Manacher is a signal
of depth, not usually a requirement.

### 🧪 Codeforces Practice Set
- [ ] CF 1326D1 — Prefix-Suffix Palindrome (Easy version) (1700) — start here
- [ ] CF 1326D2 — Prefix-Suffix Palindrome (Hard version) (2200)
- [ ] CF 17E — Palisection (2900 — hard, stretch goal)

---

## 🔷 PATTERN 7 — Hashing + Segment Tree (Strings With Updates)

### 🎯 Core Idea (preview)
Manacher and static-suffix tricks break the moment a character can change. The fix: store two
hashes per segment-tree node — the forward hash and the reverse hash of that segment — and merge
them the same way you'd concatenate strings. A substring is a palindrome exactly when its forward
hash equals its reverse hash.

### 🧩 CSES Problems
- [ ] Palindrome Queries *(point update + "is substring [l,r] a palindrome?")*

> ⚠️ Honest note: this is a narrow, niche combo — there isn't a rich independent CF problem set
> for it. Master it directly through the CSES problem, then move on. It's a great "build it once
> and understand exactly why it works" exercise rather than a grind-many-problems pattern.

---

## 🔷 PATTERN 8 — Suffix Array + LCP Array (Kasai's Algorithm)

### 🎯 Core Idea (preview)
Sort ALL suffixes of the string lexicographically. Once sorted, most "tell me something about
every substring" questions (count distinct substrings, kth smallest substring, longest repeated
substring) become questions about the **Longest Common Prefix (LCP)** between *adjacent* suffixes
in that sorted order — an O(n) or O(n log n) computation via Kasai's algorithm, instead of
comparing every pair of substrings directly.

### 🧩 CSES Problems
- [ ] String Functions *(build Z-array, prefix function, suffix array, and LCP array — the setup problem)*
- [ ] Inverse Suffix Array *(realize this is just inverting a permutation — the "trick" problem)*
- [ ] Distinct Substrings *(sum of `n - sa[i] - lcp[i]` over all suffixes)*
- [ ] Repeating Substring *(the answer is just `max(lcp array)`)*
- [ ] Substring Order I *(kth smallest DISTINCT substring — binary search over suffixes + LCP)*
- [ ] Substring Order II *(kth smallest substring counting multiplicity — needs a segment tree on top)*
- [ ] Substring Distribution *(distinct-substring count broken down by length, via difference array on LCP)*

### 💼 Product-Company Relevance
*Longest Duplicate Substring* (LeetCode) is literally "find the max LCP between adjacent sorted
suffixes" — but in interviews it's far more commonly solved with **binary search on answer length +
rolling hash**, not a full suffix array build. Building the full suffix array is rarely required
live, but understanding *why* the binary-search-plus-hashing shortcut works requires this mental
model.

### 🧪 Codeforces Practice Set
- [ ] CF "Yet Another LCP Problem" (`string suffix structures`, `data structures`) — direct LCP-array application
- [ ] CSES cluster above is genuinely your primary training ground for this pattern

> ⚠️ Honest note: this is the most CF-sparse pattern in this whole list — high-quality standalone
> suffix-array problems on CF tend to be Div. 1 hard (2200+) and bundle in segment trees, DSU, or
> trees. Don't force 15 problems here. Nail the 7 CSES problems deeply — that alone puts you ahead
> of almost everyone who's only done LeetCode.

---

## 🔷 PATTERN 9 — Booth's Algorithm / String Rotations (BWT)

### 🎯 Core Idea (preview)
To find the lexicographically smallest rotation of a string in O(n) (instead of comparing all n
rotations pairwise), Booth's algorithm runs a KMP-style failure-function scan over the string
doubled with itself. `String Transform` flips this idea around: it's the **inverse Burrows-Wheeler
Transform** — reconstruct the original string purely from the sorted-rotations' last-character
column.

### 🧩 CSES Problems
- [ ] Minimal Rotation
- [ ] String Transform *(inverse BWT)*

### 💼 Product-Company Relevance
Rare direct ask. Occasionally shows up as "find the smallest string among all rotations" — usually
solvable interview-side with a simpler O(n log n) doubling technique rather than true Booth's.

### 🧪 Codeforces Practice Set
> ⚠️ No strong dedicated CF problem set exists for this specific technique — it's a short,
> self-contained algorithm best learned by implementing it directly against the 2 CSES problems.

---

## 🔶 BONUS PATTERN 10 — Counting DP: Distinct Subsequences

### 🎯 Core Idea (preview)
This is **not** a string-structure algorithm — it's a DP where `dp[i]` = number of distinct
subsequences using the first `i` characters, and the trick is subtracting off the double-count
using **the last occurrence of the current character**. Good to know it's a different beast from
patterns 1–9 so you don't go hunting for a "string algorithm" that doesn't exist here.

### 🧩 CSES Problems
- [ ] Distinct Subsequences

### 💼 Product-Company Relevance
*Distinct Subsequences II* (LeetCode) is the exact same problem. Legitimately asked as-is.

---
---

## ✅ Part 3 — How We'll Actually Run This

1. We do **Pattern 1 → Pattern 10**, in order, no skipping.
2. For each pattern: I teach the core idea conversationally (no templates to memorize) → you pick
   1–2 CSES problems from that section → you attempt → we discuss → then CF practice problems →
   then you check the boxes above → then we move to the next pattern.
3. Once all 10 are done and CSES is fully green, we do a **mixed review round**: I'll pull random
   problems across all 9 real patterns (no labels, cold) to simulate contest/interview conditions.
4. **Only after that** do we move to LeetCode, applying the same patterns to company-tagged problems.

> Ready when you are — say "start Pattern 1" and we begin with String Hashing.
