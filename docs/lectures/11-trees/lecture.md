---
title: "Trees"
subtitle: "DSA27 — Lecture 11 · Data Structures and Algorithms"
author: "Dr. Haitham A. El-Ghareeb"
institute: "Faculty of Computers and Information Sciences, Mansoura University"
date: "Week 11 · Fall 2026"
lang: en
---

::: {.handout-only}

> **How to read this document.** This is the handout for Lecture 11. It holds
> everything on the slides, plus what I said out loud. Until now every structure
> was a **line**: each element had one next element. A tree lets each element
> have **two**. That one change gives a structure that is fast to search *and*
> fast to change — provided it stays short. Every operation is taught the same
> way — the idea, one step, the code, the whole run, the cost — on **one tree**,
> so that you learn the pattern once and then only the differences. Boxes
> marked **Deeper** hold the parts that are examined but are not needed to
> follow the lecture; read them second.
>
> Slides: `DSA27-L11-slides.pdf` · Code: `dsa/tree.py` ·
> Tests: `tests/test_tree.py` ·
> Interactive demo: [Tree Lab](https://helghareeb.github.io/DSA27/lectures/11-trees/tree-lab.html) ·
> Practise online: [Week 11 quiz and written practice](https://helghareeb.github.io/en/quizzes/dsa27/week-11/) (in [Arabic](https://helghareeb.github.io/ar/quizzes/dsa27/week-11/))

:::

## From lines to branches

Every structure so far was **linear**: array, linked list, stack, queue.

- A sorted array: search $O(\log n)$, but insert **$O(n)$** — everything shifts.
- A linked list: insert $O(1)$ once you are there, but search **$O(n)$**.

Today: a structure where search **and** insert cost $O(\log n)$ — **while it
stays short**.

::: {.handout-only}

Lecture 08 ended with a trade-off. A sorted array is wonderful to search and
miserable to change, because an insertion in the middle shifts every element
after it. A linked list is the opposite. The **binary search tree** combines
the two: it is a linked structure, so an insertion changes a couple of
references and moves nothing; and its links are arranged so that every
comparison discards a whole branch, the way binary search discards half an
array. The small print, which is the second half of this lecture: the tree is
only fast while it is **short**. Feed it sorted data and it grows into a linked
list. You will measure exactly that.

*Tree*: شجرة. *Binary search tree*: شجرة البحث الثنائية.

:::

## Today: one tree, one pattern

T = the BST built by inserting `8, 3, 10, 1, 6, 14, 4, 7, 13`.

Every operation today, on T, in the same five beats:

1. **Idea** — one sentence
2. **One step** — a picture
3. **Code** — short, plain Python from `dsa/tree.py`
4. **The whole run** — predict first, then look
5. **Cost** — in terms of the **height** h

Search, insert, min and max, delete, the four walks — and what happens when the tree is a line.

::: {.handout-only}

Use the same tree for everything, on paper and in the Tree Lab. Because the
tree never changes, the differences you see are the differences between the
operations. The question bank uses a second tree, **Q**, for practice, so that
you meet every operation twice.

:::

# The Vocabulary

## A tree, and its words

![](figures/terminology.png){width=90%}

::: {.handout-only}

A **tree** is a set of **nodes** joined by **edges**, with one special node,
the **root**, such that every other node has **exactly one parent** and there
is exactly one path from the root to it. *Node*: عقدة. *Edge*: حافة. Trees are
drawn upside down: the root at the top, the leaves at the bottom.

| Term | Meaning | Arabic |
|---|---|---|
| root | the one node with no parent | الجذر |
| parent, child | an edge goes from a parent down to a child | الأب، الابن |
| siblings | children of the same parent | الإخوة |
| leaf | a node with **no** children | ورقة |
| internal node | a node with at least one child | عقدة داخلية |
| subtree of x | x and everything below it | شجرة فرعية |
| ancestor, descendant | anything on the path above x; anything below x | سلف، خَلَف |

The figure's tree has 9 nodes and 8 edges. That is always so: every node except
the root has exactly one edge coming down into it, so **n nodes have $n - 1$
edges**. The **subtree** idea is what makes trees recursive: the left child of
the root is itself the root of a tree, and so on, down to the empty tree.

Trees are everywhere you see a hierarchy: folders inside folders, the HTML of a
web page, an expression such as `(3 + 4) * 2` as your compiler sees it, a
tournament bracket, and the call tree of `fib(n)` in Lecture 03. When an
element may have **several** parents, or paths go round in a circle, it is a
**graph** — Week 14.

:::

## Depth and height

- **Depth** of a node: edges from the **root** down to it. The root has depth 0.
- **Height** of a node: edges on the longest path from it **down** to a leaf.
  A leaf has height 0.
- **Height of the tree** = height of the root. Empty tree: **$-1$**.

| | depth | height |
|---|---|---|
| measured from | the root, down | the leaves, up |
| the root | 0 | the tree's height |
| a leaf | its level | 0 |

::: {.handout-only}

*Depth*: العمق. *Height*: الارتفاع. In T, 6 has depth 2 (8 $\rightarrow$ 3 $\rightarrow$ 6) and height 1
(6 $\rightarrow$ 4). The tree's height is 3, along 8 $\rightarrow$ 10
$\rightarrow$ 14 $\rightarrow$ 13. Nodes of the same depth form a **level**
(مستوى).

**Why $-1$ for the empty tree?** So that the recursive definition works with no
special case: the height of a node is $1 + \max(\text{left height},
\text{right height})$, and a leaf, whose two subtrees are empty, gets
$1 + \max(-1, -1) = 0$. `dsa/tree.py` counts **edges**, never nodes, and the
tests check `-1` for an empty tree and `0` for one node. Some books count
nodes, so the same tree is one taller there; in an exam, say which you mean.
Height matters more than anything else this week, because it is the cost of a
search.

:::

## Binary trees: at most two children

```python
class TreeNode:
    __slots__ = ("value", "left", "right")

    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left                # Lecture 05's Node had one link;
        self.right = right              # this one has two
```

- Level d holds at most $2^d$ nodes, so a tree of height h holds at most $2^{h+1} - 1$.
- n nodes need height **at least $\lfloor \log_2 n \rfloor$** — and may have height up to **$n - 1$**.

::: {.handout-only}

*Binary tree*: شجرة ثنائية. `TreeNode` is **given** in `dsa/tree.py`, exactly
as above. An empty subtree is simply `None`, as the end of a linked list was,
and the tree object keeps one reference, `self.root`, as `LinkedList` kept
`self.head`. Nothing else is stored — not a size, not a list of values: the
storage rule of Lecture 02. `dsa/tree.py` also gives `to_edges(node)`, so you
can **see** your tree before any traversal works:

```python
from viz.draw import draw_tree
from dsa.tree import BinarySearchTree, to_edges
bst = BinarySearchTree([8, 3, 10, 1, 6, 14, 4, 7, 13])
draw_tree(to_edges(bst.root), highlight={"6"})
```

| n | shortest possible height | tallest possible height |
|---|---|---|
| 7 | 2 | 6 |
| 1,000 | 9 | 999 |
| 1,000,000 | 19 | 999,999 |

The lower bound is the promise: a million values can live in a tree of height
19, and then a search needs at most 20 comparisons — exactly binary search. The
upper bound is the threat: the same values can also form a single chain. Which
one you get depends, for this week's tree, on the **order in which the values
arrive**.

> **Deeper — three named shapes, and the sum.** Summing the levels 0 to h gives
> $1 + 2 + \dots + 2^h = 2^{h+1} - 1$, the doubling sum of Lecture 04. To hold n
> nodes you need $2^{h+1} - 1 \ge n$, so $h \ge \lfloor \log_2 n \rfloor$. A
> tree is **full** when every node has 0 or 2 children; **perfect** when every
> level is completely filled ($2^{h+1} - 1$ nodes); **complete** when every
> level is full except possibly the last, which is filled from the left. The
> heap of Week 12 is a complete binary tree — which is what lets it live in a
> plain `Array` with no links at all.

:::

# Binary Search Trees

## The BST property

![](figures/bst-property.png){width=94%}

For **every** node x: everything in x's **left subtree** is smaller than x, and
everything in its **right subtree** is larger.

::: {.handout-only}

That is the invariant of a **binary search tree** (BST). Note the words
*subtree* and *everything*: it is **not** enough that the left child is smaller
and the right child larger. In the right-hand tree, every parent–child pair
looks fine — 3 < 8, 10 > 8, 9 > 3 — but 9 sits in 8's **left** subtree, where
nothing may be larger than 8. Search that tree for 9 and you go right at 8, and
never find it.

The purple ranges in the left-hand tree are the right way to think about it.
Each node inherits an allowed range from its ancestors: going left from 8
promises "less than 8"; going right from 3 then adds "greater than 3". So 6 must
lie in (3, 8), and 7, below it on the right, in (6, 8).

**Duplicates.** `dsa/tree.py` keeps values unique: inserting a value already in
the tree leaves it unchanged (`test_duplicates_are_ignored`).

:::

## `is_valid`: carry the range down

```python
def is_valid(self):
    return self._is_valid(self.root, None, None)     # no limits at the root

def _is_valid(self, node, low, high):
    if node is None:
        return True                                  # the empty tree is valid
    if low is not None and node.value <= low:
        return False
    if high is not None and node.value >= high:
        return False
    return (self._is_valid(node.left, low, node.value)     # left: new high
            and self._is_valid(node.right, node.value, high))  # right: new low
```

**Common mistake:** checking only `left.value < node.value < right.value` — the broken tree above passes.

::: {.handout-only}

The recursive helper takes a node and the range its ancestors allow (`None`
means "no bound"). Going left from x, the new high is x; going right, the new
low is x. Each node is checked once: $O(n)$. An alternative is to use the
in-order walk of later in this lecture — a tree is a BST exactly when its
in-order walk is strictly increasing — but the range version shows **why**.

:::

## Search: one path down

![](figures/search.png){width=60%}

```python
node = self.root
while node is not None:
    if value == node.value:
        return True
    node = node.left if value < node.value else node.right
return False
```

::: {.handout-only}

**Idea:** compare, then go left or right — every comparison rules out a whole
subtree. This is the body of `contains`. It is the linked list's traversal loop
from Lecture 05 with one change: at each node there are two ways on, and the
comparison chooses. The invariant makes it safe: if 7 is anywhere, and 7 < 8, it
can only be in 8's left subtree.

The loop ends in one of two ways: it finds the value, or it steps onto `None`,
the empty subtree where the value **would** have been. In the right-hand panel,
the search for 5 goes 8, 3, 6, 4, and then to the right of 4, which is empty.
That place matters: it is exactly where `insert(5)` will put the new node.

**Cost.** One comparison per level, at most h + 1 levels: $O(h)$. A loop, so no
call stack: $O(1)$ extra space.

:::

## Insert: a search that fails

![](figures/insert.png){width=66%}

**Idea:** walk down as in search; where the way on is `None`, hang the new node **there**.
A new value always becomes a **leaf**. Nothing moves: $O(h)$.

::: {.handout-only}

Compare with Lecture 02's sorted array: finding the place was $O(\log n)$, but
making room shifted everything after it, $O(n)$. Here the place is found with the
same halving idea, and making room costs nothing at all — one reference changes
from `None` to the new node.

:::

## Insert: the code

::: {.slides-only}

```{=latex}
\footnotesize
```

:::

```python
def insert(self, value):
    new = TreeNode(value)
    if self.root is None:
        self.root = new                     # an empty tree: the new node IS the root
        return
    node = self.root
    while True:
        if value == node.value:
            return                          # already there: ignore
        if value < node.value:
            if node.left is None:           # stop AT the parent ...
                node.left = new             # ... while you can still set its link
                return
            node = node.left
        else:
            if node.right is None:
                node.right = new
                return
            node = node.right
```

**Common mistake:** walking until `node is None`, then `node = TreeNode(value)` — the tree never sees it.

::: {.handout-only}

You must stop **at the parent**, while you can still set its `left` or
`right`. Walk until `node is None` and you have lost the parent: assigning
`node = TreeNode(value)` only rebinds a local variable. Either look one step
ahead, as here, or keep a `parent` variable as you walk.
`test_structure_is_actually_a_bst` checks the **shape** after inserting 8, 3,
10: an implementation that kept a sorted Python list and pretended to be a tree
would pass `in_order` and fail this.

:::

## Min and max: no comparisons at all

```python
def min(self):
    if self.root is None:
        raise ValueError("min of an empty tree")
    node = self.root
    while node.left is not None:         # go left for as long as you can
        node = node.left
    return node.value
```

`max` is the mirror image: go **right**. $O(h)$, and not a single comparison of values.

::: {.handout-only}

Everything smaller than a node is to its left, so the smallest value is the node
you reach by going left for as long as you can. The invariant has already done
the comparing, when the values were inserted. In T, min is 8 $\rightarrow$ 3
$\rightarrow$ 1, and max is 8 $\rightarrow$ 10 $\rightarrow$ 14. That same walk,
started at a node's **right child**, finds the smallest value larger than that
node: its **in-order successor**. Delete needs exactly that.

:::

# Delete: Three Cases

## Find it, then look at its children

| The node has… | Do this |
|---|---|
| no children (a **leaf**) | the parent's link becomes `None` |
| **one** child | the parent's link points at that child |
| **two** children | copy in the **in-order successor**, then delete the successor |

![](figures/delete-easy.png){width=62%}

::: {.handout-only}

Deleting is the hard operation of the week: removing a node must leave a tree
that is still a BST, and a node with two children cannot simply vanish, because
two subtrees would be left hanging from one link. Like insert, delete must hold
on to the **parent** of the node it removes — and the parent may not exist,
when the node is the root; then `self.root` itself changes.

**A leaf** (delete 1): `3.left = None`. **One child** (delete 14): 14's only
child is 13, so point 14's parent, 10, straight at it: `10.right = 13`. The
leaf case is the one-child case with a child of `None`: "point the parent at
the child" works for both.

:::

## Two children: the in-order successor

**Predict:** `delete(3)` — 3 has two children. Which value takes its place?

::: {.slides-only}

. . .

:::

![](figures/delete-successor.png){width=96%}

::: {.handout-only}

**4.** A node with two children cannot be removed, so we remove a **different**
node and move its value up. The value must be larger than everything left of 3
and smaller than everything right of it: the **in-order successor**, the
smallest value in 3's right subtree — go right once, to 6; then left for as
long as possible, to 4. Copy 4 into the node that held 3, then delete the old
4. The result is a BST: 4 is larger than everything on the left (those were
smaller than 3) and smaller than everything left on the right (4 was the
smallest of them).

**Why the second delete is always easy.** The successor was reached by going
left until there was no left child — so it **has no left child**. It is a leaf
or has one child on the right: one of the two easy cases, never a third.

> **Deeper — the bottom row, where students trip.** Deleting the root 8: go
> right once, to 10; 10 has no left child, so **10 itself** is the successor —
> the "go left" loop runs zero times. Copy 10 into the root. The old 10 has a
> right child, 14, so the second delete is the one-child case, and the link that
> changes is the deleted node's own `right`, not a `left` further down. A
> delete that always writes `successor_parent.left = ...` passes the top row
> and corrupts this one — `test_delete_the_root` is exactly this case.
> `dsa/tree.py` uses the successor (Hibbard, 1962); the **predecessor**, the
> largest value of the left subtree, would work equally well.

:::

## Delete: the code

::: {.slides-only}

```{=latex}
\footnotesize
```

:::

```python
def delete(self, value):
    parent, node = None, self.root
    while node is not None and node.value != value:    # 1. find it, and its parent
        parent = node
        node = node.left if value < node.value else node.right
    if node is None:
        return                                         # not there: nothing to do
    if node.left is not None and node.right is not None:
        parent, successor = node, node.right           # 2. two children: right once,
        while successor.left is not None:              #    then left as far as possible
            parent, successor = successor, successor.left
        node.value = successor.value                   #    copy it up ...
        node = successor                               #    ... and delete IT instead
    child = node.left if node.left is not None else node.right  # 3. at most one child
    if parent is None:
        self.root = child                              # it was the root
    elif parent.left is node:
        parent.left = child
    else:
        parent.right = child
```

**Common mistake:** forgetting the root — when `parent is None`, it is `self.root` that changes.

::: {.handout-only}

The three cases collapse into two. The two-children case does its copy and then
**hands over** to the easy case, by moving `node` and `parent` down to the
successor. Step 3 is written once and serves all three cases. To decide which
link of the parent to change, compare **identities**: `parent.left is node` asks
"which reference points at this node?". Everything costs one walk down: to find
the node, then on down to the successor — $O(h)$ in total. The tests delete the
root (`test_delete_the_root`) and delete every node in turn until the tree is
empty (`test_delete_everything`).

> **Deeper — the recursive form.** A helper `_delete(node, value)` that returns
> the new root of the subtree, so that the caller writes
> `node.left = self._delete(node.left, value)`, is the other classic form. It is
> elegant and correct — and it recurses h deep, which the loop does not. Either
> is accepted in the lab.

:::

# The Four Traversals

## Recursion on trees

A **traversal** visits **every** node once. Every recursive walk has the same skeleton:

- **base case:** the empty tree — do nothing
- **recursive case:** walk the left and right subtrees, and do something with the node

*Where* you put "do something" gives three orders:

| Name | Order | Mnemonic |
|---|---|---|
| **pre-order** | node, left, right | node **before** its subtrees |
| **in-order** | left, node, right | node **in between** |
| **post-order** | left, right, node | node **after** its subtrees |

::: {.handout-only}

*Traversal*: اجتياز الشجرة. Why is the skeleton correct? In the language of Lecture 03: the base case,
`None`, needs no recursion; each recursive call is on a **subtree**, strictly
smaller than the tree, so the recursion always reaches the base case; and if the
calls on the two subtrees do their job, adding the node itself completes the job
for the whole tree. That is induction on the size of the tree. The fourth walk,
**level-order**, is a different kind of walk and needs a queue.

:::

## In-order: the code

```python
def in_order(self):
    out = []                            # one list for the whole walk
    self._in_order(self.root, out)
    return out

def _in_order(self, node, out):
    if node is not None:
        self._in_order(node.left, out)
        out.append(node.value)
        self._in_order(node.right, out)
```

Move `out.append` **up** one line $\rightarrow$ pre-order. **Down** one line $\rightarrow$ post-order.

::: {.handout-only}

This is the reference `in_order` exactly. The list `out` is passed down so that
every call appends to the **same** list: $O(n)$ appends in total. The
traversals **return a new list** — output for the caller; the tree never keeps
it (`test_traversals_return_a_fresh_list`). Building one list beats returning
lists and concatenating them: `left_list + [node.value] + right_list` copies at
every level, and on a tall tree that is $O(n^2)$ — the slicing trap of Lectures
03 and 08 in a new form.

:::

## Four walks of one tree

**Predict:** what is the **in-order** walk of T?

::: {.slides-only}

. . .

:::

![](figures/traversals.png){width=84%}

::: {.handout-only}

**Sorted**: 1, 3, 4, 6, 7, 8, 10, 13, 14. Check each walk against its rule
before you trust the picture. Pre-order starts at the root, then does the
**whole** left subtree (3, 1, 6, 4, 7) before touching the right. Post-order
finishes both subtrees of a node before the node: the root comes **last**.
Level-order reads the drawing row by row. These four lists are the four
traversal tests in `tests/test_tree.py`.

**Cost.** Every walk visits each node once: **$O(n)$ time**. The recursive
three use the call stack, one frame per level of the current path: **$O(h)$
space**.

:::

## What each one is FOR

| Walk | Gives you | Used for |
|---|---|---|
| **in-order** | a BST's values **sorted** | listing, range queries, `is_valid` |
| **pre-order** | parents before children | **saving** a tree: re-inserting it **rebuilds** the same shape |
| **post-order** | children before parents | **freeing** a tree; **evaluating**; sizes and heights |
| **level-order** | nearest to the root first | printing by level; shortest distances (BFS, Week 14) |

::: {.handout-only}

**In-order is sorted** — for a BST, and only for a BST. That is the most useful
property a BST has: a sorted listing in $O(n)$ with no sorting at all. (Insert n
values, then walk in-order: that is **tree sort**, $O(n \log n)$ when the tree
stays short, and $O(n^2)$ when it does not.)

**Pre-order rebuilds.** Insert the pre-order list of a BST into an empty BST and
you get the same shape back: each node arrives after its parent, so it follows
the same path and lands in the same place
(`test_pre_order_rebuilds_the_same_tree`). The in-order list would not do — it
is sorted, and inserting sorted values builds a chain.

**Post-order frees and evaluates.** A node is visited only after both its
subtrees are done. In C, where you free memory yourself, that is the only safe
order: free the parent first and you lose the links to its children. It is also
how anything that **depends on the children** is computed — `size` and `height`
are post-order walks.

:::

## Post-order evaluates: expression trees

![](figures/expression.png){width=84%}

::: {.handout-only}

An **expression tree** has an operator at each internal node and a number at
each leaf. The tree for `(3 + 4) * 2` has `*` at the root, because `*` is done
**last**: the shape records the precedence, and the tree needs no brackets.
Walk it three ways and you get the three notations of Lecture 06:

- **post-order**, `3 4 + 2 *`, is **postfix** — exactly what `evaluate_postfix`
  evaluated with a stack. Evaluating the tree is also post-order: the left
  operand, then the right, then the operator — 7, then 14.
- **pre-order**, `* + 3 4 2`, is **prefix** (Polish) notation.
- **in-order**, `3 + 4 * 2`, is **infix** — and it is **wrong**: it reads as 11.
  In-order loses the brackets.

Week 15 builds these trees from text with a recursive parser.

:::

## Level-order needs a queue

```python
def level_order(self):
    out = []
    if self.root is None:
        return out
    queue = CircularQueue()             # YOUR queue from Week 7
    queue.enqueue(self.root)
    while not queue.is_empty():
        node = queue.dequeue()          # first in, first out
        out.append(node.value)
        if node.left is not None:
            queue.enqueue(node.left)
        if node.right is not None:
            queue.enqueue(node.right)
    return out
```

**Common mistake:** a **stack** instead of a queue — you get a depth-first walk.

::: {.handout-only}

Level-order (**breadth-first**) cannot be written with the recursive skeleton,
because it does not finish one subtree before starting the other: after
visiting 3 it must visit 10, in the **other** subtree, before 3's children.
Recursion follows the call stack, last in, first out; to visit in order of
**distance from the root** you need the nodes in the order they were **found**:
first in, first out. A queue. Replace it by a stack and the walk becomes
depth-first — pre-order, if you push the right child before the left. Week 14
puts the two side by side on graphs, as BFS and DFS.

![](figures/level-order.png){width=40%}

The queue must be **your** `CircularQueue` from Week 7, not a Python list — and
it must **grow**: a tree's widest level can hold about half its nodes (a perfect
tree of 31 nodes has 16 leaves) and the default capacity is 8
(`test_level_order_on_a_wide_tree`). A list with `pop(0)` would give the right
answer in $O(n)$ per dequeue, $O(n^2)$ for the walk: Lecture 07's `SlowQueue`.
**Cost:** $O(n)$ time; the queue holds at most one level and a bit of the next,
$O(w)$ space for the widest level w.

:::

## Height and size: post-order in one line

```python
def _height(self, node):
    if node is None:
        return -1
    return 1 + max(self._height(node.left), self._height(node.right))
```

`size` is the same shape: 0 for `None`, else $1 +$ left size $+$ right size.

::: {.handout-only}

Both calls must finish before the `max` can be taken — the node is handled
**after** its children: a post-order walk, $O(n)$. The base case returns $-1$,
so a leaf comes out as 0 with no special case. `size` is counted, not stored;
you could keep a counter up to date in `insert` and `delete`, but then `delete`
must decrement only when it really removed something.

:::

# When the Tree Is a Linked List

## Same values, same code, different height

**Predict:** insert `1, 2, 3, 4, 5, 6, 7` in this order. What shape do you get?

::: {.slides-only}

. . .

:::

![](figures/degenerate.png){width=80%}

A **chain**: each new value is the largest so far, so it always goes right. Height $n - 1$. Every operation: **$O(n)$**.

::: {.handout-only}

"Insert 1, 2, 3, 4, 5 in order and you build a linked list wearing a tree
costume." Nothing in the code is wrong; the BST property holds
(`test_degenerate_tree_is_a_linked_list`). But the tree has one node per level,
and a search for the largest value visits every node. Sorted input is not
exotic: dates in a log, student IDs in registration order, a file saved sorted
last time. The same values inserted middle-first — 4, then 2 and 6, then the
rest — give the shortest tree 7 nodes can make. If the values arrive in a
**random** order, the tree is short on average: the average depth of a node is
about $2 \ln n \approx 1.39 \log_2 n$ (Hibbard, 1962) — but "random" is a
promise about your data that you usually cannot make.

:::

## Measured

![](figures/measured.png){width=94%}

::: {.handout-only}

Real measurement of the reference `dsa/tree.py`, for n = 64 to 4,096 values.
**Left: height.** Sorted insertion gives exactly $n - 1$: 4,095 for 4,096
values. Random insertion order (the mean over 20 shuffles) gives 26.6 at
n = 4,096 — more than the 12 of a perfect tree, but a **logarithm**: it grows
by 2 to 3 each time n doubles. **Right: time** for one successful `contains`:
at n = 4,096, about 205 microseconds on the chain against 1.8 on the random
tree — more than a hundred times slower, and the gap doubles every time n
doubles.

**A second failure: recursion.** The recursive `height`, `size` and traversals
go one call deeper per level. On a chain of 1,000 nodes that is 1,000 frames,
Python's default limit: `BinarySearchTree(range(1000)).height()` raises
`RecursionError`. The loops — `contains`, `insert`, `delete`, `min`, `max` —
and the queue-based `level_order` do not care. A tall tree is not only slow; it
breaks recursive code.

:::

## What each operation costs

| Operation | Cost | Short tree | Chain |
|---|---|---|---|
| `contains`, `insert`, `delete`, `min`, `max` | $O(h)$ | $O(\log n)$ | $O(n)$ |
| `height`, `size`, `is_valid`, the four walks | $O(n)$ | | |
| extra space, recursive walks | $O(h)$ | $O(\log n)$ | $O(n)$ |
| extra space, `level_order` | $O(w)$ | up to $O(n)$ | $O(1)$ |

**The cure** — balanced trees (AVL, red-black) that keep h at $O(\log n)$ — is **beyond this course**.

::: {.handout-only}

h is the height and w the width of the widest level. A **short** BST is the
first structure of the course to give $O(\log n)$ for search **and** insert
**and** delete, and it keeps the values in order — so min, max, the sorted
listing and range queries come for free. A hash table (Week 13) will beat it on
plain lookups but cannot list its keys in order.

> **Deeper — balanced trees.** A balanced search tree (شجرة متوازنة) keeps a
> little extra information in each node — a height, or a colour — and after an
> insertion or deletion repairs the shape with **rotations**, local
> rearrangements of two or three nodes that keep the BST property and change
> the heights. **AVL trees** (Adelson-Velsky and Landis, 1962) keep the two
> subtrees of every node within one level of each other; **red-black trees**
> (Guibas and Sedgewick, 1978) allow a little more slack for fewer repairs.
> Both guarantee height $O(\log n)$. None of the bylaws of this course asks for
> them (`docs/course/02-coverage.md`), so they are named here, not examined.
> Java's `TreeMap` and C++'s `std::map` are red-black trees. What you **can**
> do with this week's tree: if you have all the values up front, sort them and
> insert the middle first, then the middles of each half — height
> $\lfloor \log_2 n \rfloor$. That is W11-C4.

:::

# This Week

## Exercises, and the Tree Lab

| Method | Target | The trap |
|---|---|---|
| `insert` | $O(h)$ | stop **at the parent**; ignore duplicates |
| `contains`, `min`, `max` | $O(h)$ | a loop; `ValueError` on empty |
| `delete` | $O(h)$ | keep the parent; the **root**; successor = right child |
| `height`, `size` | $O(n)$ | empty tree is $-1$ and 0 |
| `is_valid` | $O(n)$ | carry **(low, high)** down, not just the parent |
| the four traversals | $O(n)$ | return a **new list**; level-order uses **your** queue |

```powershell
pytest tests/test_tree.py -v
```

[**Tree Lab**](https://helghareeb.github.io/DSA27/lectures/11-trees/tree-lab.html): every operation of today, line by line, on T or on your own values.

::: {.handout-only}

Twenty-six tests. `TreeNode`, `to_edges`, `__init__` and the dunders (`len`,
`in`, `iter`, `repr`) are given; the dunders call your methods, so they start
working as soon as those do. Write `insert` and `contains` first — every other
test builds its tree with `insert`. Then draw your tree with `to_edges` and
`draw_tree` before going further. `level_order` imports `CircularQueue` from
`dsa/queue.py`: your Week 7 queue, and it must grow when full. Do not filter
with `-k tree`: `-k` matches the file name `test_tree.py`, so it selects all 26.
Filter by the method instead: `-k delete`, `-k order`.

**The Tree Lab** runs the code of this lecture one line at a time: the tree,
the current node, the path so far, the variables, the call stack of the
recursive walks, the queue of level-order, and the output list. Insert, search
and delete your own values; build the sorted chain and watch the height; switch
on **What happens next?** to answer before each step; and run each operation's
**common mistake**.

- **Online:** <https://helghareeb.github.io/DSA27/lectures/11-trees/tree-lab.html>
- **Offline:** open `docs/lectures/11-trees/tree-lab.html` from your copy of the
  repository. It is a single file and needs no installation.

:::

## Homework 11 — before Lecture 12

1. **Implement** `dsa/tree.py` until all 26 tests pass.
2. **Draw** the BST built by inserting 50, 30, 70, 20, 40, 60, 80, 35, 45, 65,
   then delete 20, 30 and 50 — the tree after each delete.
3. **Walk** your tree of item 2 (before the deletes) in all four orders, by hand.
   Check with your code.
4. **Measure** the height of a BST built from `range(n)` and from a shuffled
   `range(n)`, for n up to 900, in `notebooks/11-trees.ipynb`. Why stop at 900?

Then: the [**Week 11 quiz**](https://helghareeb.github.io/en/quizzes/dsa27/week-11/) — 22 questions and 5 written ones, every answer explained ([Arabic](https://helghareeb.github.io/ar/quizzes/dsa27/week-11/)).

::: {.handout-only}

For item 2, the three deletes are one of each case: 20 is a leaf, 30 then has
one child, and 50 is the root with two children. Draw the successor search for
50 before you move anything. For item 4, the answer to "why 900" is the
recursion limit — try 1,000 and read the error. Then explain why `contains`
would still work on that tree. Do the quiz **after** the homework, as a check,
not instead of it.

:::

## Seven things to keep

1. A tree: one root; every other node has **one parent**; n nodes, $n - 1$ edges.
2. **Height** counts edges down to the deepest leaf; empty tree $-1$, leaf 0.
3. BST: **left subtree smaller, right subtree larger** — for the whole subtree.
4. Search, insert, min, max: **one path down**, $O(h)$.
5. Delete with two children: copy the **in-order successor**, delete it instead.
6. In-order is **sorted**; pre-order **rebuilds**; post-order **frees and
   evaluates**; level-order needs a **queue**.
7. Sorted input gives height **$n - 1$**: $O(n)$. Balanced trees fix it — beyond
   this course.

**Next — Week 12:** heaps — a **complete** binary tree stored in an `Array`, with no links at all.

::: {.handout-only}

A BST keeps **everything** in order, and pays for it with a height that the
input order controls. A heap keeps only a weaker order — every parent is smaller
than its children — and in exchange its shape is always complete, so its height
is always $\lfloor \log_2 n \rfloor$, whatever the input. It cannot search for an
arbitrary value quickly, but it always knows its minimum: exactly what a
priority queue needs.

---

## Sources and further reading

- **D. E. Knuth.** *The Art of Computer Programming*, vol. 1, *Fundamental
  Algorithms*, 3rd ed., Addison-Wesley, 1997, §2.3 "Trees" — the vocabulary,
  and traversals; vol. 3, *Sorting and Searching*, 2nd ed., 1998, §6.2.2
  "Binary tree searching" — insertion, deletion, and the average depth of a
  random BST.
- **T. N. Hibbard.** "Some combinatorial properties of certain trees with
  applications to searching and sorting", *Journal of the ACM* 9(1), 1962 —
  deletion by the successor, and the analysis of random BSTs.
- **T. H. Cormen, C. E. Leiserson, R. L. Rivest and C. Stein.** *Introduction to
  Algorithms*, 3rd ed., MIT Press, 2009, chapter 12, "Binary search trees".
- **M. T. Goodrich, R. Tamassia and M. H. Goldwasser.** *Data Structures and
  Algorithms in Python*, Wiley, 2013, chapters 8 "Trees" and 11 "Search trees".
- **G. M. Adelson-Velsky and E. M. Landis.** "An algorithm for the organization
  of information", *Soviet Mathematics Doklady* 3, 1962 — AVL trees.
- **L. J. Guibas and R. Sedgewick.** "A dichromatic framework for balanced
  trees", *Proceedings of the 19th FOCS*, 1978 — red-black trees.

Every figure in this lecture is generated by `tools/figures_l11.py`. The
diagrams are fixed trees, checked against the reference solution; the measured
figure times a working `dsa/tree.py` and will differ slightly on your machine.

:::
