
# 🔹 1. FIFO (First-In First-Out)

## 🧠 Idea

Remove the page that entered memory **earliest**.

Think of it like a **queue**:

- Insert at rear
- Remove from front

---

## ⚙️ Data Structures

- Queue (or circular buffer)

---

## 🔁 Working

1. If page is already in memory → do nothing
2. If not:
    - If space available → insert
    - Else → remove front page → insert new page

---

## 🧾 Pseudo-code

``` cpp
initialize queue Q
initialize set S  // for fast lookup

for each page p:
    if p in S:
        continue   // hit

    if size(Q) < capacity:
        Q.push(p)
        S.insert(p)
    else:
        victim = Q.front()
        Q.pop()
        S.remove(victim)

        Q.push(p)
        S.insert(p)

```

---

## ❗ Key Insight

- Ignores usage pattern
- Can remove frequently used pages

---

# 🔹 2. LRU (Least Recently Used)

## 🧠 Idea

Remove the page that was **not used for the longest time**

---

## ⚙️ Data Structures

Best implementation:

- **HashMap + Doubly Linked List**

Why?

- O(1) access
- O(1) update

---

## 🔁 Working

- Whenever a page is used → move it to front (most recent)
- Remove from back (least recent)

---

## 🧾 Pseudocode

```cpp
initialize DLL (most recent at front)
initialize map M (page -> node)

for each page p:
    if p in M:
        move node of p to front
    else:
        if size == capacity:
            victim = DLL.back()
            remove victim from DLL
            M.erase(victim)

        insert p at front
        M[p] = pointer to node
```

---

## ❗ Key Insight

- Very efficient in practice
- Expensive if implemented naïvely (timestamps)