Assignment 5 Report
---------------------

# Team Members

Linus Ihringer
Sebastian Jung


# GitHub link to your (forked) repository

https://github.com/ihringerlinus/Assignment5

# Task 1

Note: Some questions require you to take screenshots. In that case, please join the screenshots and indicate in your answer which image refer to which screenshot.

1. What happens when Raft starts? Explain the process of electing a leader in the first
term.

Ans: When Raft starts, all servers begin in the follower state. Each server sets a randomized election timeout. If a follower does not receive any communication from a leader within this timeout, it transitions to the candidate state and initiates an election by incrementing its term and sending RequestVote messages to other servers. The other servers respond with their votes. If a candidate receives votes from a majority of the servers, it becomes the leader for that term. The leader then starts sending heartbeat messages (AppendEntries) to maintain its authority and prevent new elections.

2. Perform one request on the leader, wait until the leader is committed by all servers. Pause the simulation.
Then perform a new request on the leader. Take a screenshot, stop the leader and then resume the simulation.
Once, there is a new leader, perform a new request and then resume the previous leader. Once, this new request is committed by all servers, pause the simulation and take a screenshot. Explain what happened?

Ans: The first request on the leader was successfully processed and committed by all servers, ensuring data consistency across the cluster. When the first leader was paused, a new leader election took place among the remaining servers. The new leader then processed the second request. Upon resuming the previous leader, it recognized the new leader's authority and synchronized its state accordingly. The final request was committed by all servers, demonstrating Raft's ability to maintain consistency and handle leader changes seamlessly.

3. Using the same visualization, stop the current leader and two additional servers. After a few increments, pause the simulation and take a screenshot. Then resume all servers and restart the simulation. After the leader election, pause the simulation and take a screenshot. How do you explain the behavior of the system during the above exercise?

Ans: Because a majority of servers (3 of 5) were stopped, the remaining servers could not elect a new leader. As a result, the system was unable to process any requests during this period. As soon as all servers were resumed, a new leader election took place among the active 5 servers, allowing the system to regain its functionality and process requests again. This exercise highlights Raft's reliance on majority consensus for leader election and operation.

# Task 2

1. Which server is the leader? Can there be multiple leaders? Justify your answer using the statuses from the different servers.
Ans: After starting all three servers only one node is the leader in the admin/status endpoint. In Raft, there cannot be multiple leaders at the same time because the protocol is designed to ensure that only one server can act as the leader in a given term. This is achieved through the election process, where a candidate must receive votes from a majority of the servers to become the leader. If multiple servers were to claim leadership simultaneously, it would lead to inconsistencies and conflicts in the log replication process. The statuses from the different servers confirm that only one server holds the leader role while the others are followers.

2. Perform a PUT operation for the key "a" on the leader. Check the status of the different nodes. What changes have occurred and why (if any)?

Ans: The leader returns 204 No Content indicating that the PUT operation was successful. When checking the status of the different nodes, the leader's log will show the new entry for key "a" with its corresponding value, while the follower nodes will have replicated this entry in their logs as well. The term and commit index may also be updated to reflect the new state of the log. These changes occur because Raft ensures that log entries are replicated across all servers to maintain consistency.

3. Perform an APPEND operation for the key "a" on the leader. Check the status of the different nodes. What changes have occurred and why (if any)?

Ans: The leader returns 204 No Content indicating that the APPEND operation was successful. When checking the status of the different nodes, the leader's log will show the updated entry for key "a" with the appended value, while the follower nodes will have replicated this updated entry in their logs as well. The term and commit index may also be updated to reflect the new state of the log. These changes occur because Raft ensures that log entries are replicated across all servers to maintain consistency. This because APPEND is also a raft command

4. Perform a GET operation for the key "a" on the leader. Check the status of the different nodes. What changes have occured and why (if any)?

Ans: There were no changes in the status of the different nodes after performing a GET operation for the key "a" on the leader. The GET operation is a read-only operation and does not modify the state of the log or the data stored in the servers. Therefore, the term, commit index, and log entries remain unchanged across all nodes. The leader simply retrieves the value associated with key "a" without affecting the overall state of the Raft cluster.



# Task 3

1. Shut down the server that acts as a leader. Report the status changes that you get from the servers that remain active after shutting down the leader. What is the new leader (if any)?

Ans:
* **Status Changes:**
    * **Before:** node3 was `leader`, the others were `followers`.
    * **After:** The original leader is `unreachable`. node3 has transitioned to `leader` state. The **Term** has incremented by 1.
* **Explanation:** When the leader shuts down, the followers stop receiving heartbeats. One follower times out first, becomes a `candidate`, increments the term, and requests votes. Since 2 out of 3 nodes are still active (a majority), it receives enough votes to become the new leader.

1. Perform a PUT operation for the key "a" on the new leader. Then, restart the previous leader, and indicate the changes in status for the three servers. Indicate the result of a GET operation for the key "a" to the previous leader.

Ans:
* **Status Changes (after restart):**
    * **Before Restart:** The cluster had 2 active nodes (1 leader, 1 follower).
    * **After Restart:** The old leader rejoins. It detects the higher term from the current leader and immediately transitions to the `follower` state. It synchronizes its log to match the current leader.
* **GET Operation:** A GET request to the previous leader (now a follower) returns the new value (e.g., `["The", "Test"]`). This confirms it has successfully updated its log and state.

3. Has the PUT operation been replicated? Indicate which steps lead to a new election and which ones do not. Justify your answer using the statuses returned by the servers.

Ans:
* **Yes, it has been replicated.**
* **Justification:** For the PUT operation to return a success code (204) on the new leader, it must have been committed. In Raft, a commit only happens if a majority of nodes (here, 2 out of 3: the new leader and the remaining follower) have stored the entry.
* **Election Steps:**
    1.  Leader crashes $\rightarrow$ Heartbeats stop.
    2.  Follower Timeout $\rightarrow$ State changes to `candidate` $\rightarrow$ Term increments.
    3.  Candidate requests votes $\rightarrow$ Receives vote from the other active node.
    4.  Majority reached $\rightarrow$ Candidate becomes `leader`.

4. Shut down two servers: first shut down a server that is not the leader, then shut down the leader. Report the status changes of the remaining server and explain what happened.

Ans:
* **Status Changes:**
    * **Before:** Cluster was healthy with 3 nodes.
    * **After:** Only 1 node remains active. This node loses its leader status (if it was leader) or remains a `candidate`/`follower`. The **Term** might keep increasing if it tries to start elections, but it never transitions to a stable `leader` state because it cannot form a quorum (1 < 3/2 + 1).

5. Can you perform GET, PUT, or APPEND operations in this system state? Justify your answer.

Ans:
* **No.**
* **Reason:** Raft requires a majority (Quorum) to commit any state change (PUT/APPEND). With only 1 active node out of 3, no majority exists. GET operations also typically fail or block to prevent serving stale data (network partition check).

6. Restart the servers and note down the changes in status. Describe what happened.

Ans:
* **Status Changes:**
    * **After Restart:** All nodes come back online. They exchange term and log information. A single `leader` is established (either the one with the highest term/log takes over, or a new election happens). All nodes eventually synchronize their logs and the cluster status becomes healthy (1 Leader, 2 Followers).

## Network Partition

For the first experiment, create a network partition with 2 servers (including the leader)
on the first partition and the 3 other servers on the other one. Indicate the changes that occur in the status of a server on the first partition and a server on the second partition. Reconnect the partitions and indicate what happens. What are the similarities and differences between the implementation of Raft used by your key/value service (based on the PySyncObj library) and the one shown in the Secret Lives of Data illustration from Task 1? How do you justify the differences?

Ans:
* **Status Changes:**
    * **Partition 1 (Minority, Old Leader):** The leader cannot receive acknowledgments from a majority. It might remain in `leader` state temporarily but cannot commit new logs.
    * **Partition 2 (Majority):** Nodes time out due to missing heartbeats. A new election occurs. A new `leader` is elected because 3 nodes form a majority of 5. The term increments.
* **Reconnect:**
    * The old leader (Part 1) sees the new leader's higher term.
    * It steps down and becomes a `follower`.
    * Logs synchronize to the state of the new leader.
* **Comparison:** This behavior matches the *Secret Lives of Data* visualization perfectly. The minority partition pauses processing, and the majority partition continues with a new leader. Upon reconnection, the minority joins the new term.

For the second experiment, create a network partition with 3 servers (including the
leader) on the first partition and the 2 other servers on the other one. Indicate the changes that occur in the status of a server on the first partition and a server on the second partition. Reconnect the partitions and indicate what happens. How does the implementation of Raft used by your key/value service (based on the PySyncObj library) compare to the one shown in the Secret Lives of Data illustration from Task 1?

Ans: 
* **Status Changes:**
    * **Partition 1 (Majority, Old Leader):** The system continues to function normally. The leader has a quorum (3/5) and can commit operations.
    * **Partition 2 (Minority):** Nodes receive no heartbeats. They become `candidates` and start elections. Since they cannot reach a majority, they split votes or fail, repeatedly timing out and incrementing their **Term** (Term Inflation).
* **Reconnect:**
    * The nodes from Partition 2 rejoin with a much higher Term number than the current leader.
    * **Observation:** The current stable leader sees the higher term, assumes its own term is outdated, and steps down to `follower`. This forces a new election, even though the main cluster was healthy.
    * **Comparison:** This matches the behavior in *Secret Lives of Data* (and standard Raft without Pre-Vote), where isolated nodes can disrupt the active cluster upon reconnection due to high terms.


# Task 4

1. Raft uses a leader election algorithm based on randomized timeouts and majority voting, but other leader election algorithms exist. One of them is the bully algorithm, which is described in Section 5.4.1 of the Distributed Systems book by van Steen and Tanenbaum. Imagine you update the PySyncObject library to use the bully algorithm for Raft (as described in the Distributed Systems book) instead of randomized timeouts and majority voting. What would happen in the first network partition from Task 3?

Ans:
* **Scenario:** Network partition splits the cluster into a group of 2 (with the old leader) and a group of 3.
* **Bully Algorithm Behavior:**
    * In **Partition 1 (Minority)**: The node with the highest ID in this group declares itself leader.
    * In **Partition 2 (Majority)**: The node with the highest ID in this group *also* declares itself leader.
* **Result:** **Split-Brain**. There would be two active leaders independent of each other, potentially accepting conflicting writes. Raft avoids this because the minority group can never gather enough votes to elect or maintain a leader.

2. Why is it that Raft cannot handle Byzantine failure? Explain in your own words

Ans: 
* **Reason:** Raft assumes a **Fail-Stop** model (nodes work correctly or stop working). It does not account for malicious or buggy nodes ("Byzantine" faults) that send incorrect information.
* **Explanation:** In Raft, followers trust the leader's messages blindly if the Term and Index match. A Byzantine node could:
    * Lie about having stored a log entry.
    * Send conflicting log entries to different followers.
    * Corrupt the log.
    * Raft lacks mechanisms like cryptographic signatures or Byzantine fault-tolerant consensus (like PBFT) to detect and reject such malicious behavior.
