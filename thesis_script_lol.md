# Speaker Script for Raft Verification Presentation

## 0.1 Title Slide
Good morning/afternoon everyone. 
My name is Akhoury Shauryam, and today I'll be presenting our work on "Can Inductive Invariants be used to enhance efficiency of Bounded Model Checking," with a case study on the Raft Consensus Protocol. 
This research was conducted with my colleague Shobhit Singh under the guidance of Professor M. Praveen and Professor M.K. Srivas at Chennai Mathematical Institute.

## 0.2 Overview
In this presentation, I'll walk you through our motivation for this work, our goals and contributions, some preliminaries on model checking, the Raft consensus protocol itself, our modeling approaches, the specific bugs we targeted, our verification properties, and finally our experimental results.

## 1.1 Motivation - A toy example
Let's start with a simple example to illustrate the problem. 
Here we have a toy program that implements a left-shift operation on an array of 8 registers. 
The correct implementation shifts values to the left and copies v[0] to v[7]. 
The buggy implementation allows a non-deterministic choice between v[0] and v[7] for the last position.

The invariant states that after every 8 shifts, we should return to the original state. 
The LTL property specifies that whenever v[0] is 0, the next value of v[0] should be 1. 
This simple example helps demonstrate the verification techniques we'll discuss.

## 1.2 Traces
Here's a visual representation of how the bug manifests. 
The arrows show the trace where incorrect behavior emerges, leading to a state where the invariant is violated. 
As you can see, after a series of shifts, we end up with the array [7,0,1,2,3,4,5,6] instead of returning to the original state.

## 1.3 Techniques to catch the bug
There are several techniques we can use to catch this kind of bug:
- Symbolic Model Checking (SMC) can find all bugs but suffers from state explosion
- Bounded Model Checking (BMC) is fast for shallow bugs but requires specifying a bound
- K-Induction can provide full proofs if k is small enough

Our focus is on combining BMC with invariants to improve efficiency.

## 1.4 Verification Techniques — Quick Scan
Let's quickly compare the verification techniques. 
SMC finds all bugs but suffers from state explosion. 
BMC is fast for shallow bugs but requires knowing the depth. 
K-Induction can provide full proofs with small k but becomes unwieldy for longer properties. 
Our proposed approach, BMC with invariants, can cut runtime but requires crafting appropriate invariants.

## 1.5 Adding Invariants → Faster BMC
The core idea is to inject auxiliary inductive invariants that help guide the verification. 
These invariants act as "trip-wires" - the solver may hit any one counter-example sooner, leading to smaller unwind bounds and reduced runtime. 
If one or more invariants fail, safety is likely to fail as well.

Mathematically, we're looking for invariants Inv₁, Inv₂, ..., Invₘ such that each invariant implies the safety property. 
This way, a violation of any invariant signals a potential safety issue.

## 1.6 Adding Invariants ⇒ Faster BMC (8-register Rotator)
Going back to our register example, we add an auxiliary invariant stating that all register values must be distinct. 
This property is initially true and remains true in the correct implementation. 
With this extra property, we get an early violation signal without needing to unroll to the full depth.

Since the values are distinct initially (0 through 7), and should remain distinct in proper operation, checking this invariant allows us to catch the bug more efficiently than waiting for the full 8N steps to check the main safety property.

## Moving on to Goals and Contributions

## 2.1 Project Goal
Our main goal is to apply inductive-invariant-aided BMC to speed up formal verification and debugging of a realistic consensus protocol model. 
We chose Raft's Leader Election Phase due to its practical importance. 
The challenge is state explosion due to network and timing complexities. 
Our hypothesis is that carefully chosen invariants can lead to earlier counter-examples and shorter bounds in k-induction.

## 2.2 Contribution 1 — Abstract Raft‐LEP Model
Our first contribution is an abstract model of Raft's Leader Election Protocol. 
We aimed for a balanced abstraction - detailed enough to capture the election logic but abstract enough to be tractable in model checkers like NuSMV and CBMC. 
We used non-determinism to model message delays and losses, avoiding explicit channel state explosion.

## 2.3 Contribution 2 — Safety Property & Inductive Invariants
Our second contribution focuses on the safety property and inductive invariants. 
The target safety property is ensuring at most one leader per term. 
We crafted several inductive invariants to support this:
- Unique Quorum: There can only be one node with majority votes
- Unique Vote: A voter grants at most one vote per term
- Leader Uniqueness: If two nodes think they're leaders, their terms must differ

## 2.4 Contribution 3 — Proof Experiments
For our third contribution, we conducted various proof experiments:
1. We verified the correct model with no counter-examples
2. We injected a bug by dropping the "unique vote" rule
3. We ran experiments on the buggy model using BMC alone and BMC with invariants
4. We collected metrics like SAT calls, maximum bound k, and wall-clock time

## Moving on to Preliminaries

## 3.1 Model Checking
Model checking is an automated technique to verify whether a system satisfies a temporal logic specification. 
The system is modeled as a transition system with states, initial states, and transition relations. 
Formally, we represent it as M = (S, I, T) where S is the set of states, I defines initial states, and T is the transition relation.

## 3.2 BMC vs SMC
There are two main approaches to model checking:
- Symbolic Model Checking uses Binary Decision Diagrams to represent states and transitions, exploring the full reachable state space
- Bounded Model Checking unrolls the system to a bounded depth and searches for counterexamples by encoding the problem as a SAT/SMT formula

## 3.3 Bounded Model Checking (BMC): Encoding
BMC works by constructing a logical formula that encodes the initial state, transition relations up to bound k, and the negation of the desired property. 
If this formula is satisfiable, a counterexample exists within k steps. 
If unsatisfiable, no violation exists within the bound, but we cannot conclude correctness beyond that bound.

The formula takes the form: I(s₀) ∧ ∧ᵢ₌₀ᵏ⁻¹ T(sᵢ, sᵢ₊₁) ∧ ∨ᵢ₌₀ᵏ ¬φ(sᵢ)

## 3.4 Normal Induction vs K-Induction
Standard induction proves a property holds in all reachable states by showing it holds in the initial state and is preserved by transitions. 
K-induction extends this by proving a property holds over sequences of k transitions, which can be more powerful when the property is not directly inductive.

## 3.5 Over-Approximation in K-Induction
K-induction assumes the property holds for k steps and then checks if it holds at step k+1. 
However, it doesn't require that the initial state in this sequence to be reachable, so it proves the property over a superset of reachable traces. 
However, iff the property holds on this over-approximation, it must hold on the actual reachable states.

## 3.6 Effect of Increasing k
As we increase k in k-induction, we get a stronger inductive hypothesis, tighter approximation of reachable states, and eliminate more spurious counterexamples.

## Moving on to What is Raft

## 4.1 What is Distributed Consensus?
Distributed consensus ensures multiple nodes in a distributed system agree on a common value despite failures. 
Challenges include network partitions, node failures, and balancing consistency with availability. 
Common use cases include replicated state machines and coordination services like ZooKeeper, etcd, and Consul.

## 4.2 Introduction to Raft
Raft is a consensus algorithm designed to be understandable, consistent, and fault-tolerant. 
It ensures distributed state machines apply the same sequence of commands even with failures, though it doesn't protect against Byzantine failures - only server crashes, network timeouts, message drops, etc. 
Byzantine failures involve nodes that can behave arbitrarily or maliciously, which Raft does not address.

## 4.3 Introduction to Raft (continued)
Raft decomposes consensus into three subproblems:
1. Leader Election: Choosing a single leader to manage the log
2. Log Replication: Ensuring logs on all servers are consistent
3. Safety: Maintaining the correctness of the log

Every change to the system state goes through the leader.

## 4.4 Raft Server States and Persistent State
A Raft server can be in one of three states:
- Leader: Handles all client interactions
- Follower: Passive state, responds to requests
- Candidate: Temporary state to initiate elections

## 4.5 Overview of Leader Election Process
The leader election process follows this flow: Heartbeat Timeout → Candidate → RequestVote → Majority → Leader → Leader sends heartbeats. 
The key guarantee is Election Safety: ensuring a unique leader per term.

## 4.6 What is a Term?
Raft divides operations into phases, with each normal operation phase beginning with an election. 
Each node keeps a local counter called Term which increases monotonically. 
Terms help detect stale leaders and RPCs. 
If a server sees a higher term, it updates its currentTerm and becomes a follower.

## 4.7 Triggering an Election
Followers start an election after a timeout. 
The transition steps include incrementing the current term, becoming a candidate, voting for itself, and sending RequestVote RPCs to all servers.

## 4.8 The RequestVote RPC Explained
The RequestVote RPC includes the term, candidateId, lastLogIndex, and lastLogTerm. 
A vote is granted if the term is at least the current term, the follower hasn't voted yet, and the candidate's log is at least as up-to-date. 
When a majority votes, the candidate becomes the Leader.

## 4.9 Election Outcomes and Split Votes
An election can have three outcomes:
- Win: Candidate gets majority and becomes Leader
- Lose: Candidate receives a message with equal or higher term
- Split Vote: No majority due to simultaneous candidates

Here RAFT employs randomized heartbeat timeouts, where followers wait a randomly generated set amount of time before they become a candidate, this reduces split votes.

## 4.10 Election Safety Properties
The Election Safety property ensures only one leader is elected per term. 
This is enforced by allowing only one vote per server per term and requiring a majority for leadership. 
With these rules, it's impossible to have two leaders in the same term.

## 4.11 Election Timeline
This slide shows a typical election timeline when a leader crashes and is replaced:
1. A is Leader in term 3
2. A crashes
3. B times out, becomes a Candidate in term 4
4. B gets votes from C and D
5. B becomes Leader
6. B sends heartbeats to all nodes
7. E updates to term 4
8. A recovers
9. A receives a heartbeat with term 4
10. A steps down to Follower

## 4.12 Objectives
Our objectives were to:
1. Create a model for the Leader Election Protocol and prove its correctness using K-Induction
2. Inject a duplicate vote bug in the model and try to catch it efficiently using Bounded Model Checking

## Moving on to Models with Different Levels of Abstraction

## 5.1 What is a reasonable abstract model?
The key question we addressed was: "What is a reasonable abstract model that allows invariants proved for an abstract model to expedite BMC at a concrete implementation level?" 
This took about 50% of our research time to determine.

## 5.2 Network Diagram
This diagram illustrates the network structure in our model, showing how nodes communicate through a central network component. 
This is the most basic representation, but as we'll see, we developed several levels of abstraction.

## 5.3 Trivial Model: Level 1 Abstraction
We started with a very simple, highly non-deterministic model. 
The model includes:
- Roles (Follower, Candidate, Leader)
- Process IDs
- Terms (positive integers)
- State mapping nodes to roles and terms

The initial state has all nodes as Followers with term 1.

## 5.4 Transition Relation and LEP
The Leader Election Property (LEP) states that if two nodes are leaders, they must be in different terms. 

In our transition relation, we explicitly enforce LEP and ensure terms are non-decreasing. 
Thus, every reachable state satisfies LEP by construction since the transition relation only allows valid states where the safety property holds.
And therefore this model is correct by construction.

## 5.5 Level 2: Less Abstract Model
Our Level 2 model abstracts away network and messages but retains roles, terms, and votes to enable safety proofs. 
It includes components like roles, process IDs, terms, quorum flags, and vote tracking.

Let me detail each component of this model:
- Role = {F, C, L}: The three possible states of a node - Follower, Candidate, or Leader
- Procid = {0, 1, ..., Max}: Process identifiers for each node in the system
- Term = PosInt: Terms are positive integers that monotonically increase
- Quorum = bool: A boolean flag indicating whether a node has received a majority of votes
- Votes: [Procid] → Nat: A mapping that tracks votes received from each node
- State: Node = [Procid] → ⟨Role, Term, ms: Votes, q: Quorum⟩: The complete state mapping from process IDs to their associated role, term, vote counts, and quorum status


## 5.6 Defining LEP and Invariants
We defined the Leader Election Property and supporting invariants:
- LEP: No two nodes can be leaders in the same term, and a leader must have a quorum
- UniqueQ: At most one node has quorum
- MajQ: A node has quorum only if it received a majority
- UniqueVote: No double voting in a term

## 5.7 Transition System: T(s, s')
The transition system specifies the initial state where all nodes are followers with term 1 and no votes. 
The transitions enforce rules like terms never decreasing, candidates with quorum becoming leaders, the quorum rule requiring a majority, vote monotonicity, and the unique vote rule.

## 5.8 Refinement and Existential Abstraction
We use existential abstraction, where a concrete system refines an abstract system if every concrete transition maps to an abstract transition. 
This ensures every concrete trace is representable in the abstract model. 
The refinement conditions include initialization, simulation, and conformance. 
Initialization ensures the abstract initial states include all concrete initial states.
Simulation guarantees that concrete transitions can be mapped to abstract transitions.
Conformance verifies that concrete state sequences satisfy abstract properties.

## 5.9 Criteria for Abstract Models
Our criteria for good abstract models include:
1. Sufficient state information to exhibit safety violations and inductive invariants
2. Existential abstraction that doesn't exclude correct implementation behavior
3. Non-inductive safety property, reducing invariant complexity
4. Simplicity to ease verification effort

## 5.10 Level 3: Concrete Model
While Level 2 abstraction was useful for conceptual understanding, it wasn't detailed enough to model the specific bugs we wanted to target, especially the duplicate vote bug. 
This necessitated developing a more concrete model with explicit message handling and network behavior.
Our most concrete model represents Raft as a distributed system with nodes and a message queue. 
Messages include type, payload, sender, sender's term, and receiver. 
All message sending and delivery are non-deterministic, and the network may delay messages arbitrarily but is fair.

## 5.11 Node Local State
Each node maintains state including its term, role, log, votedFor, timeout, votes received, inbox, and outbox. 
Node behavior is defined by transition rules reacting to inbox messages and timeouts.

## 5.12 Network Diagram
This slide visualizes the network structure with nodes, inboxes, and outboxes, showing how messages flow between nodes through the network.
The network model at this level is quite complex, with multiple message types, queues, and non-deterministic delivery.
This complexity makes BMC struggle to complete verification in reasonable time frames, as the state space explosion becomes significant even for small cluster sizes.

## 5.13 Level 2.5: Working model
In our working model, we removed the explicit network module. 
Instead of modeling message queues explicitly, messages are generated non-deterministically and placed directly in node inboxes, where the inbox size is exactly 1.
At each transition, a node is selected, a valid message is generated based on the current system state, and the node processes it.

## 5.14 Message Generation and Processing Flow
The flow follows these steps:
1. Choose a node
2. Generate a valid message consistent with the node's role and term
3. Process the message and update the node's state variables
4. Repeat for the next step

## 5.15 Node State
Each node maintains its term, role, votedFor, and votes received. 
We also track the global set of terms across all nodes.

## 5.16 Message Schema
Messages are tuples with a type (Heartbeat, VoteRequest, VoteGrant), sender, and term. 
There's no explicit queueing or delivery delays - messages exist virtually and must be justified by the current state.

## 5.17 Network Diagram (Working Model)
This shows a simplified view of our network model with nodes and inboxes, removing the central network component and focusing on the inbox processing.

## 5.18 Inbox Constraint Refinement Process
The most important thing that is required to get this model to work is finding a correct Constraint on the Inbox.
To do this, we've described the method in the flow chart
So, we refined the constraints on the inbox through an iterative process:
1. We run the model
2. When a property fails, analyze the trace, this part got easier once we removed all the avoidable loops in our code implementation.
3. Read the trace and understand why its spurious, add constraints to rule out these unrealistic behaviors
4. Repeat this until the bug free model passes the safety check via K-Induction and let the buggy model fail.

## 5.19 Follower: State Transitions
Apart from the constraints on the inbox, we have further rules on the transitions for each role.
Followers have these state transitions:
- On timeout with no message: Become a candidate, increment term, vote for self, and reset votes
- On vote request: Reset timeout, update votedFor if valid.
- On heartbeat with valid term: Reset timeout, update term if higher, reset vote

## 5.20 Candidate: State Transitions
The transitions for candidates are:
- On vote grant: Update vote counters, become leader if majority reached
- On heartbeat with higher term: Become follower, update term
- On vote request with higher term: Become follower, update term, change votedFor to sender.

## 5.21 Leader: State Transitions
And similarly, the leaders transition on:
- Heartbeat with higher term: Become follower, update term
- Vote request with higher term: Become follower, update term, change votedFor to sender
- Vote grant: Update vote counters

## Now we can move on to Targeted Bugs

## 6.1 Common Bugs in Raft Implementations
Despite Raft's design for simplicity and understandability; implementations still have subtle bugs in areas like vote counting, term handling, log indexing, and recovery management.
This can arise due to the gap between Raft's conceptual simplicity and the complexity of implementing it in real distributed systems with asynchronous networks, concurrent operations, and partial failures.
We focused on the duplicate vote bug commonly known as raft-45.

## 6.2 Duplicate Vote Bug (raft-45)
The description of raft-45 is:
"Candidates accept duplicate votes from the same follower in the same election term.
(A follower might resend votes because it believed that an earlier vote was dropped
by the network). Upon receiving the duplicate vote, the candidate counts it as a new
vote and steps up to leader before it actually achieved a quorum of votes"

And so, the duplicate vote bug occurs when candidates accept duplicate votes from the same follower in the same election term. 
This violates leader safety, allowing two leaders to be elected in the same term, potentially leading to data inconsistency and other violations.

## 6.3 Modeling Duplicate Votes
We implemented two vote counters:
- true_votes: Counts the number of unique voters (correct behavior)
- fake_votes: Counts total votes including duplicates (buggy behavior)
We can toggle between them with a compilation flag -DINJECT_DUPLICATE_VOTE_BUG in our CBMC code, allowing us to selectively enable the bug.

## Moving on to Verification Properties

## 7.1 Safety Properties for Raft
We defined several safety properties:
- check_safety_property(): No two leaders in the same term
- check_leader(): Leaders must have quorum of votes
- check_unique_vote(): No node votes twice in the same term
- check_unique_quorum(): Only one node has majority in same term

## 7.2 LEP-Safe Abstraction for Verification
We verify LEP via BMC by checking if the concrete transitions imply the negation of the abstract safety properties and invariants. 
This approach leverages our abstraction to make verification more efficient.
The key insight here is that by formally establishing a refinement relationship between our abstract and concrete models, we can show that any property proven in the abstract model also holds in the concrete model.
This allows us to craft properties and invariants at the abstract level where reasoning is simpler, then apply them to verify the more complex concrete model.
By focusing on the essential state variables that affect leader election safety, we can drastically reduce the verification complexity.

## 7.3 Using Abstraction to Verify LEP
We check if the concrete transitions (when unrolled to bound k) imply the negation of the conjunction of LEP and the supporting invariants. 
If this implication holds, we've found a counterexample.
Specifically, we construct a formula of the form:
T_cs(cs_0, cs_1) ∧ ... ∧ T_cs(cs_k, cs_k+1) ⇒ ¬[LEP(A(cs)) ∧ UniqueQ(A(cs)) ∧ MajQ(A(cs)) ∧ UniqueVote(A(cs))]
Where T_cs represents the concrete transition relation, cs_i are concrete states, A is the abstraction function, and the right side contains our properties and invariants.
This approach allows us to find violations in the concrete model that manifest as violations of our abstract invariants, often with fewer BMC steps than directly checking the concrete safety property.
Here our concerned A is just a simple projection function.

## 7.4 Checking Model with CBMC
We used CBMC (C Bounded Model Checker) with various flags to check our model, including specifying properties, enabling traces, injecting bugs, and using fixed initialization.
The key flags we used include:
- --property main.assertion.X: This allows us to target specific assertions in the code
- --trace-hex: Produces a detailed trace output when violations are found
- -DINJECT_DUPLICATE_VOTE_BUG: Enables our duplicate vote bug implementation
- -DUSE_FIXED_INIT: Forces a specific initial state to focus the verification

## 7.5 CBMC-style Pseudocode
This part of the code is just to visualize the main loop of the CBMC code.
The pseudocode looks like this:

init_states();
__CPROVER_assume(check_property());
where we intialize the states

for (int k = 0; k < varK; k++) {
    apply_transition();
    __CPROVER_assume(check_property());
}
which means we enter the loop and assume the property we are concerned holds good with after each transition

apply_transition();

here we take a final transition before asserting our safety property

assert(check_safety_property());

In summary or verification approach initializes states, assumes properties hold, applies transitions while assuming properties, and then checks if the safety property holds after the final transition.

## Now, we can finally move on to the Experimentation and the Results

## 8.1 Goals
Our experimental goals were:
1. Compare performance of BMC for Safety Property vs BMC for Invariants
2. Compare performance of Vanilla BMC vs Invariant Aided BMC
3. Compare performance of Vanilla BMC vs Constrained Search Space

## 8.2 Impossible to Find Violation with N=3
When experimenting, we noticed that we couldn't get to a violation with 3 nodes and then we realized that:
With only 3 nodes, the duplicate vote bug cannot cause a safety violation because the quorum size is 2. 
Even with a duplicate vote, the candidate would have legitimate votes from itself and one other node, which is a valid quorum. 
We need at least 4 nodes to demonstrate the bug.

## 8.3 Minimum Steps to Failure Analysis
For N=2x or N=2x+1, we need at least 2x+4 steps to find a safety violation.
For x greater or equal to 2 obviously as described before.
So this violation happens when,
a Node A times out and becomes a candidate and votes for itself
another Node B, recieves a vote request from Node A generated into its inbox, and chooses to processes it via changing its votedFor
then Node A can have Node B's voteGrant generated into its inbox, and this can happen a total of x times.
After which Node A has recieved a total of x+1 votes, and becomes the leader.
Now a similar thing can happen with Nodes C and Node D, and in totality this will require 2x+4 steps to get to the violation of our safety property
This makes it easier to conduct experiments, as we do not have to go one by one checking each value of K where BMC fails for the first time
but instead we can target 2x+3 and 2x+4 and conclude the smallest value of K on the basis of when it passed and when it failed

## 8.4 Goal 1: BMC for Safety Property vs BMC for Invariants
Our first experiment compared the performance of checking different invariants. 
We found that violations of the unique vote (UV) and leader uniqueness (L) invariants were detected earlier (at K=4 and K=5) than violations of the safety property (P) or unique quorum (UQ), which required K=8.
We can also see that the violation of Unique Vote and Leader Uniqueness is multiple folds faster than that of our safety property.

## 8.5 Goal 2: Vanilla BMC vs Invariant Aided BMC
Our second experiment evaluated whether assuming invariants at each step improved BMC performance. 
We found that assuming certain invariants, especially unique vote (UV), improved verification time from 210.75 seconds to 162.49 seconds.
Now this is a 23% improvement already, but we can do better.

## 8.6 Goal 3: Vanilla BMC vs Constrained Search Space
Our third experiment tested if injecting expected violations at specific steps improved performance. 
Injecting a unique vote violation at step 4 reduced verification time to 139.08 seconds, a significant improvement over both vanilla BMC and invariant-aided BMC.
Compared with our original 210 seconds, this is a 34% improvement in runtime.

## 8.7 Performance Comparison
Now, this chart visually compares the performance improvements from our different approaches for N=4 nodes. 
As you can see, injecting violations at specific points led to the best performance.

## 8.8 Modifications in the code
We also experimented with an optimization where, knowing that the Safety Property doesn't get violated until varK steps, we assume the property at the varK-1 step instead of for the whole trace. 
This makes the formula smaller while preserving the same trace.
So the code was changed to:

init_states();

for (int k = 0; k < varK; k++) {
    apply_transition();
}

here we dont have assumptions throughout the loop

__CPROVER_assume(check_property());

but we do have one assumption before the last transition
since we know that our safety property cant fail up until this point, this is helpful to us

apply_transition();

then we apply the transition and then assert the safety property

assert(check_safety_property());

Using this the SAT solver can have an easier time looking for a solution.

## 8.9 Experiments with N = 6 Nodes
With larger cluster sizes, the performance differences became more pronounced. 
Firstly all of the results take a much longer time to complete when compared to N=4, but thats because not only did the cluster size change,
but the first time we see a violation, that is, the number of steps till violation also increased
And since each transition is non-deterministic, even 2 more steps can change the number of traces to look for dramatically.
Ok so now, for N=6, vanilla BMC took 1333.11 seconds, while our best approach reduced this to 730.69 seconds.
This is a speed up of 46% in finding the violation of the bug, a huge improvement to spending more than 20 minutes on this.

## 8.10 Performance Comparison: All Assumptions
This chart shows the performance improvements for N=6 across our different approaches. 
The best approach was using a combination of helper invariants with a speed up saving around half the time.

## 8.11 Key Insights
Throughout this research, our key insights were:
- Violations to the safety property are often preceded by violations to helper invariants
- Selectively assuming helper properties can guide the model checker effectively
- Injecting violations at specific points can accelerate counterexample discovery
- Larger models are more sensitive to which invariants are assumed and when

## 8.12 Key Findings
Our key findings include:
- The duplicate vote bug can be successfully detected with BMC
- Minimum cluster size to demonstrate the bug is N=4
- Verification time grows significantly with cluster size
- Judicious selection of invariants improves performance
- There's a trade-off between tighter constraints and formula complexity

## 8.13 Conclusion
We demonstrated formal verification of Raft leader election, successfully modeled and detected the duplicate vote bug, established minimum bounds for cluster size and steps required, identified critical invariants for efficient verification, and showed that iterative constraint refinement is effective for debugging.

## 9.1 Q & A
Thank you for your attention. 
I'm now open to any questions you might have about our research.

## 9.2 Acknowledgements
Thank you again for your time. 
We'd like to express our gratitude to our advisors, Prof. M. Praveen and Prof. M.K. Srivas, and to Chennai Mathematical Institute for supporting this research. 
