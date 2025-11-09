# Trilectic Brainstorm Protocol

## Core Philosophy: 三人行必有我師
**"Among three people, there must be my teacher"** — Confucius

This protocol implements three-agent dialectical reasoning where:
- No single agent holds absolute truth
- Each perspective illuminates different facets of complex questions
- Synthesis emerges from genuine engagement with opposing views
- Role reversal in Round 10 enforces intellectual humility and detachment from positions

---

## Algorithm Structure

### Variable Definitions
```
no_round            // number of dialectical rounds (default: 10)
agent[role]         // agent role assignment
agent.A             // Agent A: Thesis Agent (initial position advocate)
agent.B             // Agent B: Antithesis Agent (counter-position advocate)
agent.C             // Agent C: Synthesis Agent (integrative philosopher)
shared_file         // brainstorm markdown file (file-based state sharing)
line_count          // current file line count (change detection mechanism)
```

### Configuration
```
no_round = 10       // 10 rounds allows depth: structure → anchoring → dissolution → synthesis
                    // can be redefined per session based on topic complexity
```

### Main Loop: Sequential Read-Respond-Append Pattern
```
for i in no_round:
	// ROUND 10 SPECIAL CASE: Role Reversal
	// Purpose: Enforce detachment from positions, prevent ego-identification with views
	// Agent A becomes Agent B, Agent B becomes Agent A, Agent C remains unchanged
	if i = 9:  // Round 10 (0-indexed)
		swap(agent.A, agent.B)

	// ═══════════════════════════════════════════════════════════════
	// AGENT A: Thesis Agent (Position Advocate)
	// ═══════════════════════════════════════════════════════════════
	if agent.A:
		// READ: Monitor file state via line-count tracking
		prev_line_count = line_count
		line_count = count_lines(shared_file)

		// RESPOND: Generate thesis based on round number
		if i = 0:
			// Round 1: Initial position (no prior synthesis to respond to)
			thesis_content = generate_thesis(topic)
		else:
			// Subsequent rounds: Respond to previous synthesis
			// This creates evolutionary depth as focus shifts through rounds
			wait_for_synthesis(round[i-1])  // async wait, goto loop if unavailable
			synthesis_prev = read_synthesis(round[i-1])
			thesis_content = respond_to(synthesis_prev)

		// APPEND: Write thesis to shared file (atomic file-based state update)
		append(shared_file, thesis_content) as thesis[i]
		line_count = count_lines(shared_file)  // update change tracker

	// ═══════════════════════════════════════════════════════════════
	// AGENT B: Antithesis Agent (Counter-Position Advocate)
	// ═══════════════════════════════════════════════════════════════
	if agent.B:
		// READ: Wait for thesis in current round
		wait_for_thesis(round[i])  // async wait, goto loop if unavailable
		thesis_current = read_thesis(round[i])

		// RESPOND: Generate counter-position
		antithesis_content = challenge(thesis_current)

		// APPEND: Write antithesis to shared file
		append(shared_file, antithesis_content) as antithesis[i]
		line_count = count_lines(shared_file)  // update change tracker

	// ═══════════════════════════════════════════════════════════════
	// AGENT C: Synthesis Agent (Integrative Philosopher)
	// ═══════════════════════════════════════════════════════════════
	else:  // agent.C (final step in each round)
		// READ: Wait for BOTH thesis AND antithesis in current round
		// Critical: Agent C must not proceed until both Agent A and Agent B complete
		wait_for_both(thesis[i], antithesis[i])  // async wait, goto loop if unavailable
		thesis_current = read_thesis(round[i])
		antithesis_current = read_antithesis(round[i])

		// RESPOND: Integrate both positions
		// Special handling for Round 10: comprehensive final analysis
		if i = 9:  // Round 10
			synthesis_content = final_synthesis(
				all_rounds,           // review entire dialectic arc
				thesis_current,       // current thesis (from former Agent B)
				antithesis_current,   // current antithesis (from former Agent A)
				length = "10-minute"  // ~2000-2500 words comprehensive analysis
			)
		else:
			synthesis_content = synthesize(thesis_current, antithesis_current)

		// APPEND: Write synthesis to shared file
		append(shared_file, synthesis_content) as synthesis[i]
		line_count = count_lines(shared_file)  // update change tracker

		// GIT COMMIT: Only synthesis agent commits (round completion marker)
		// This prevents commit conflicts and clearly marks round boundaries
		git_add(shared_file)
		git_commit(message = f"Complete Round {i+1} synthesis")

		// Round complete, proceed to next iteration

// ═══════════════════════════════════════════════════════════════════════
// ERROR HANDLING: Async Wait with Sequence Enforcement
// ═══════════════════════════════════════════════════════════════════════
function wait_for_thesis(round):
	if thesis[round] not available:
		wait(10)              // 10-second async wait
		goto(start_of_loop)   // return to top of current round iteration
		                      // ensures sequence A → B → C is respected

function wait_for_synthesis(round):
	if synthesis[round] not available:
		wait(10)              // adaptive: can extend to 30-60s for longer waits
		goto(start_of_loop)

function wait_for_both(thesis, antithesis):
	if thesis not available OR antithesis not available:
		wait(10)
		goto(start_of_loop)   // Agent C must wait for both Agent A and Agent B

// ═══════════════════════════════════════════════════════════════════════
// CHANGE DETECTION: Line-Count Monitoring
// ═══════════════════════════════════════════════════════════════════════
function count_lines(file):
	// Lightweight change detection without full file read
	// Command: wc -l <file>
	// If line_count increases → new content appended → proceed
	// If line_count unchanged → wait and retry
	return system("wc -l " + file)

function detect_change():
	new_count = count_lines(shared_file)
	if new_count > line_count:
		return true  // new content available
	else:
		return false  // no change, continue waiting
```

---

## Learnings from Successful 10-Round Implementation
**Session:** Complex Philosophical Question (2025-11-09)
**Agents:** Agent A (Thesis), Agent B (Antithesis), Agent C (Synthesis)
**File:** `Brainstorm/<Topic> - Brainstorm.md`
**Status:** ✓ All 10 rounds completed successfully

### 1. File-Based State Sharing
**What worked:**
- Single shared markdown file eliminated need for inter-process communication
- Each agent reads, responds, and appends atomically
- No database, no API, no synchronization primitives needed
- Git naturally tracks full history and provides merge conflict resolution

**Implementation details:**
- File path: `Brainstorm/<Topic> - Brainstorm.md`
- Each agent has read/write access to same file
- Append-only pattern prevents overwrites
- Round headers (`## Round N`) provide clear section boundaries

**Key insight:** File system is the simplest distributed state machine for async agents.

### 2. Read-Respond-Append Pattern
**Sequential flow:**
```
Agent A:  READ(synthesis[i-1]) → RESPOND → APPEND(thesis[i])
Agent B:  READ(thesis[i])      → RESPOND → APPEND(antithesis[i])
Agent C:  READ(thesis[i] + antithesis[i]) → RESPOND → APPEND(synthesis[i]) → COMMIT
```

**Why this works:**
- Each agent waits for required input before proceeding
- Append-only prevents race conditions
- Clear dependencies: Agent B depends on Agent A, Agent C depends on both Agent A + Agent B
- Synthesis commit marks round completion

**Example from Round 3:**
- Agent A reads Round 2 synthesis → responds with refined thesis building on previous synthesis
- Agent B reads Round 3 thesis → challenges with counter-position
- Agent C reads both → synthesizes integrative framework

### 3. Loop Mechanism with Async Wait
**Adaptive wait times:**
- Start: 10-15 seconds (active development phase)
- Mid-session: 30-45 seconds (agents drafting longer responses)
- Extended: 60 seconds (awaiting human intervention or complex analysis)

**goto(start_of_loop) enforcement:**
- Prevents Agent C from proceeding without both Agent A and Agent B inputs
- Prevents Agent B from proceeding without Agent A input
- Prevents Agent A from proceeding without previous synthesis (Rounds 2-10)

**Real example (Round 10 monitoring):**
```bash
# Wait for Agent A input
sleep 15 && wc -l file.md  # 342 lines
sleep 30 && wc -l file.md  # 342 lines (no change, continue waiting)
sleep 45 && wc -l file.md  # 349 lines (Agent B added, proceed to synthesis)
```

### 4. Line-Count Change Detection
**Mechanism:**
```bash
wc -l "Brainstorm/<Topic> - Brainstorm.md"
# Output: 342 → wait → 349 → new content detected
```

**Advantages over full file reads:**
- Lightweight: O(1) vs O(n) file size
- Fast: ~5ms vs 50-500ms for large files
- Reduces context token usage in monitoring loops
- Clear signal: line count increase = new content appended

**Implementation pattern:**
```bash
prev_count=331
new_count=$(wc -l file.md | awk '{print $1}')
if [ $new_count -gt $prev_count ]; then
  tail -n +$((prev_count+1)) file.md  # read only new lines
fi
```

### 5. Git Commit on Round Completion (Agent C Only)
**Why Agent C commits:**
- Agent C completes the round (last agent in A → B → C sequence)
- Commit message documents synthesis claim and round number
- Prevents commit conflicts (only one agent commits per round)
- Git history shows clear round progression

**Commit message pattern:**
```
Complete Round N synthesis: <claim>

<brief explanation>
<key frameworks>

Status: Round N/10 complete
File: Brainstorm/<topic>.md (X lines)
```

**Benefits:**
- Each round is a git checkpoint (can roll back if needed)
- Commit history provides audit trail of dialectical evolution
- Push to remote enables multi-device agent coordination (Codespaces + local)

### 6. Round 10 Role Reversal
**Purpose:** Enforce intellectual humility and detachment from positions

**Implementation:**
- Round 1-9: Agent A (Thesis), Agent B (Antithesis), Agent C (Synthesis)
- Round 10: Agent B (Thesis), Agent A (Antithesis), Agent C (Synthesis)
- Former thesis agent must now challenge
- Former antithesis agent must now advocate

**Why this works:**
- Prevents ego-identification with "my position"
- Forces agents to steel-man the opposite view
- Tests whether positions are genuinely held or just tribal
- Final synthesis benefits from agents having inhabited both perspectives

**Real example (Round 10):**
- Agent B (former antithesis) now advocates for position they previously challenged
- Agent A (former thesis) now argues against position they initially held
- Both arguments were sharp precisely because they had to defend unfamiliar ground
- Agent C synthesis: integrative framework resolving the final tension

**三人行必有我師 in action:** By Round 10, each agent has learned from the other two by defending their positions.

### 7. Evolution of Focus Through Rounds
**Dialectical arc observed:**

**Rounds 1-3: Foundation & Framework Building**
- Establish core conceptual frameworks
- Define key distinctions and categories
- Introduce contextual nuances and edge cases
- *Purpose:* Build scaffolding for complex topic exploration

**Rounds 4-6: Deepening & Anchoring**
- Address implications and consequences
- Introduce grounding mechanisms
- Connect abstract concepts to concrete applications
- *Purpose:* Prevent conceptual drift, establish practical anchors

**Rounds 7-9: Transcendence & Integration**
- Challenge constructed frameworks
- Explore paradoxes and tensions
- Synthesize multi-phase or cyclic models
- *Purpose:* Transcend initial dichotomies without losing operational capacity

**Round 10: Comprehensive Resolution**
- Role reversal sharpens final tension
- Agent C delivers comprehensive synthesis (~2500 words)
- Resolves original question through integrative framework
- *Purpose:* Provide actionable guidance and meta-level insights

**Key insight:** Each round builds on previous synthesis, creating evolutionary depth. The protocol naturally progresses from surface to depth, theory to practice, dichotomy to integration.

### 8. Technical Patterns That Worked

**Parallel reads for context:**
```bash
# When SA needs full context, read in parallel
Read(file, offset=1, limit=30)    # Round 1 summary
Read(file, offset=300, limit=50)  # Recent rounds
Read(file, offset=340)            # Latest content
```

**Sequential waits for dependencies:**
```bash
# SA must wait sequentially, not in parallel
wait_for_thesis() && wait_for_antithesis() && synthesize()
# NOT: wait_for_thesis() || wait_for_antithesis()  # would violate sequence
```

**Adaptive polling intervals:**
- Active phase: 10-15s (rapid iteration)
- Waiting phase: 30-60s (longer drafting)
- Extended wait: Check every 60s, escalate to 120s if no activity
- *Prevents:* Excessive context token usage from tight loops

### 9. What Made This Protocol Robust

**Resilience features:**
1. **File-based state:** No single point of failure, survives agent crashes
2. **Append-only:** No overwrites, no lost data
3. **Git checkpoints:** Every round is recoverable
4. **goto(loop):** Ensures sequence respected even with timing variance
5. **Line-count:** Lightweight change detection without context bloat
6. **Role reversal:** Prevents groupthink and position calcification

**Failure modes handled:**
- File modified during edit → wait and retry
- Agent timeout → other agents continue from file state
- Round 1 integrity issue → reset and restart from stable state
- Sequence violation → goto(loop) enforces correct order

### 10. Validation: Protocol Proves Its Own Principles

**Meta-observation:**
The first deployment tested a philosophical question about the value of diverse perspectives.

**The protocol's demonstration:**
Three agents treated each other as genuine sources of information despite operating from different initial positions. By engaging in good faith—cross-checking, challenging, synthesizing—the team produced a result none could have authored alone.

**The architecture embodied the principle:** Multi-agent dialectical reasoning works because it operationalizes intellectual humility. No single perspective holds the complete truth; synthesis emerges from genuine engagement across viewpoints.

**三人行必有我師:** The protocol worked because each agent genuinely learned from the other two, round by round, culminating in a comprehensive synthesis that integrated insights from all perspectives.

---

**Protocol Status:** ✓ Validated (2025-11-09)
**Next Use:** Ready for deployment on new brainstorm topics
**Recommended:** 10 rounds for deep topics, 5-7 rounds for focused questions, 3 rounds for quick dialectical checks
