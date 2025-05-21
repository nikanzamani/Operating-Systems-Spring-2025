**Operating Systems Simulation Project**

---

## 1. Project Overview

**Goals:** Create a text-based simulator of fundamental OS functions in three incremental phases:

* **Phase 1:** CPU scheduling only.
* **Phase 2:** phase 1 + deadlock prevention.
* **Phase 3:** phase 2 + memory management (paging).

Each phase builds on the previous one. Students may choose how far to implement, but full credit requires Phase 3. Your simulator reads inputs from standard input and writes output events to standard output.

**Phases Detail**

| Phase   | Functionality                                                                                               |
| ------- | ----------------------------------------------------------------------------------------------------------- |
| Phase 1 | Preemptive CPU scheduling: `Run` and `Sleep` semantics.                                                     |
| Phase 2 | Resource allocation/deallocation (`Allocate` and `Free`) with deadlock prevention (e.g., Banker's algorithm). |
| Phase 3 | Virtual memory: (`Read` and `Write`) page tables, paging I/O penalties, and replacement policy (e.g., LRU).                    |

---


## 2. Algorithm Assignment by Team ID

Students may form teams of up to **3 students**. To determine which algorithms to implement, follow these steps:

1. Extract the **last 3 digits** from each student number.
2. Sum these values to get a number `X`. For example, a team with 2 students numbered `...999` and `...123` would compute `X = 999 + 123 = 1122`.
3. Use `X` to select the algorithms for each project phase:

### Phase 1 — CPU Scheduling Algorithm

```
X mod 4:
0 → FCFS (First-Come, First-Served)
1 → SJF  (Shortest Job First)
2 → RR   (Round Robin with quantum 5 ms)
3 → MLFQ (Multilevel Feedback Queue)
       - Queue 1: quantum 4 ms
       - Queue 2: quantum 8 ms
       - Queue 3: FCFS
```

### Phase 2 — Deadlock Strategy

```
X mod 2:
0 → Banker's Algorithm (prevent deadlocks by safe-state checks)
1 → Deadlock Detection and Recovery
       - Use a resource allocation graph to detect cycles
       - Recover by preempting resources from selected processes
```

### Phase 3 — Page Replacement Policy

```
X mod 3:
0 → LRU      (Least Recently Used)
1 → Optimum  (Simulate the best-case replacement with look-ahead)
2 → FIFO     (First-In, First-Out)
```

Be sure to clearly document your chosen algorithms in your code comments and report.

---

## 3. Input Format

Your program must parse the following input in order:

general input format as follows:
```  
n                             
m                            
a₀ a₁ a₂ … aₘ₋₁                     
PS PC
IC₀                          
<Process 0 instructions>
IC₁       
<Process 1 instructions>  
...
ICₙ₋₁
<Process n-1 instructions>    
```  


1. **Line 1:** `n` — number of processes (integer).
2. **Line 2:** `m` — number of distinct resource types (integer).
3. **Line 3:** `a₀ a₁ a₂ … aₘ₋₁` — `m` space-separated integers giving the total available instances of each resource type.

   * If `m = 0` *and* this line reads `0`, Phases 2 and 3 are skipped; only Phase 1 is active.
4. **Line 4:** `PS PC` — page size (`PS`) and total page frames (`PC`).

   * If `PS = PC = 0`, Phase 3 is skipped (only CPU scheduling + deadlock prevention in Phase 2).
5. **Process Definitions:** For each process `pid` from `0` to `n-1`, read:

   1. **Line:** `IC` — instruction count (integer).
   2. **Next `IC` lines:** Each line is one instruction, in one of the formats:

      ```
      Run <T>           # Use CPU for T ms
      Sleep <T>         # Block/sleep for T ms
      Allocate <X> <Y>  # Request X instances of resource type Y
      Free <X> <Y>      # Release X instances of resource type Y
      Read <A>          # Read logical address A
      Write <A>         # Write logical address A
      ```




---


## 4. Output Format

First, output an integer `L` — the total number of events your simulator will emit. Then output `L` lines, each describing one event. Event formats:

| Keyword | Format                             | Description                                              |
| ------- | ---------------------------------- | -------------------------------------------------------- |
| EXECUTE | `EXECUTE <pid> <start> <end>`      | Process runs on CPU from `start` to `end`.               |
| WAIT    | `WAIT    <pid> <start> <end>`      | Blocked or sleeping from `start` to `end`.               |
| GIVE    | `GIVE    <pid> <X> <res> <t>`      | Freed `X` instances of resource `<res>` at time `t`.     |
| TAKE    | `TAKE    <pid> <X> <res> <t>`      | Allocated `X` instances of resource `<res>` at time `t`. |
| DTM     | `DTM     <pid> <page> <frame> <t>` | Disk→memory page load at time `t` (start time).                       |
| MTD     | `MTD     <pid> <page> <frame> <t>` | Memory→disk page write (eviction) at time `t` (start time).           |
| RM     | `RM     <pid> <frame> <t>`        | Read from memory frame `<frame>` at time `t` (start time).            |
| WM     | `WM     <pid> <frame> <t>`        | Write to memory frame `<frame>` at time `t` (start time).             |

All indices (`pid`, resource types, pages, frames) are **zero-based**.


* For **`Read`** and **`Write`**, you must account for penalties:

  * `40ms` for disk→memory or memory→disk penalty (`DTM` and `MTD`)
  * `20ms` = in‑memory read or write penalty (`RM` and `WM`)

---


## 5. Behavioral Details  
### CPU Scheduling (Phase 1)  
- Only one process can `Run` at a time.  
- `Sleep` <T> blocks the process for exactly T ms; multiple processes may sleep concurrently.  
- Example Timeline:  


### Deadlock Prevention (Phase 2)  
- Deadlock prevention algorithm takes priority over scheduling so if the your scheduling algorithm want to run one process and the deadlock prevention doesn't allow it, you should not run that process.  
- you can take the resources from process if you return it back to them before continuing execution.

### Memory Management (Phase 3)  
- **Page Fault Handling**:  
  1. If page not in memory: Trigger `DTM` (load from disk).  
  2. If evicted page is dirty: Trigger `MTD` (write to disk).  
- for each `Read` instruction you need to have a `RM` in your report
- for each `Write` instruction you need to have a `WM` in your report
- you cant concurrently read from or write to Memory so memory commands are blocking  
- you can take resources bak from a process if you return it back before getting to the next `EXECUTION` of the process


---


## 6. Examples  
### Phase 1   
**Input**:  
```  
3  
0  
0  
0 0    
3  
Run 2  
Sleep 5
Run 3  
3  
Run 2  
Sleep 7
Run 2  
3  
Run 2  
Sleep 6
Run 3  
```  
**Output**:  
```  
9  
EXECUTE 0 0 2  
WAIT 0 2 7
EXECUTE 1 2 4  
WAIT 1 4 11
EXECUTE 2 4 6  
WAIT 2 6 12
EXECUTE 0 7 10  
EXECUTE 1 11 13  
EXECUTE 2 13 16  
```  

**Timeline:**

```
Time → 0  1  2  3  4  5  6  7  8  9  10  11  12  13  14  15 16
P0     |==E==||======W======||===E===|
P1           |==E==||=========W==========||===E===|
P2                 |==E==||=========W========|    |====E=====|
```

* there can be times when CPU is inactive (10-11)
* also there can be time where process are waiting for CPU to finish (12-13)

### Phase 2  
#### Example 1:
**Input**:  
```  
2  
1  
1  
0 0  
2  
Allocate 1 0  
Run 1  
2  
Allocate 1 0  
Run 1  
```  
**Output**:  
```  
5  
GIVE 0 1 0 0  
EXECUTE 0 0 1  
TAKE 0 1 0 1
GIVE 1 1 0 1  
EXECUTE 1 1 2   
```  


#### Example 2:
**Input**:  
```  
2  
2  
1 1  
0 0  
5  
Allocate 1 0  
Run 1
Sleep 2  
Allocate 1 1  
Run 1  
5  
Allocate 1 1  
Run 1  
Sleep 4
Allocate 1 0  
Run 1  
```  
**Output**:  
```  
14  
GIVE 0 1 0 0  
EXECUTE 0 0 1  
WAIT 0 1 3
GIVE 1 1 1 1  
EXECUTE 1 1 2   
WAIT 0 2 6
TAKE 1 1 1 3
GIVE 0 1 1 3  
EXECUTE 0 3 4  
TAKE 0 1 0 6
TAKE 0 1 1 6
GIVE 0 1 0 6  
GIVE 0 1 1 6  
EXECUTE 1 6 7  
```  


### Phase 3   
**Input**:  
```  
1  
1  
1  
100 1  
9  
Read 250
Run 5
Write 240
Read 50
Run 7
Read 30
Run 3
Read 150
Run 10

```  
**Output**:  
```  
13  
DTM 0 2 0 0  
RM 0 0 40     
EXECUTE 0 60 65
WM 0 0 65
MTD 0 2 0 85
DTM 0 0 0 125
RM 0 0 165
EXECUTE 0 185 192
RM 0 0 192
EXECUTE 0 212 215
DTM 0 1 0 215
RM 0 0 255
EXECUTE 0 275 285
```  
* page size is 100 so the logical address 0-99 are on the page 0, 100-199 are on page 1 and so on
* pay attention to the 4 reads:
  * 1st one triggers page fault and causes a `DTM` and a `RM` 
  * then we write to the same page so the 2nd `Read` will causes a `WM` and a `MTD` 
  * the 3rd `Read` is on the same page as the 2nd one so we don't need to do a `DTM`
  * last one shows that you don't need to write every page back if it isn't modified


---

## 7. Submission & Grading

* **Languages:** any; must read **stdin** and write **stdout** exactly in the specified format.
* **Testing:** automated against hidden test suites covering edge cases (e.g., simultaneous sleeps, maximum resource requests, page thrashing).
* **Grading Criteria:**

  1. **Correctness:** events match expected timelines and resource/memory invariants.
  2. **Format Adherence:** input parsing and output formatting exactly as specified.
  3. **Code Quality:** clear structure and comments explaining scheduling, allocation, and paging logic.

---


