Assignment 5 Report
---------------------

# Team Members


# GitHub link to your (forked) repository

...

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

1. Perform a PUT operation for the key "a" on the new leader. Then, restart the previous leader, and indicate the changes in status for the three servers. Indicate the result of a GET operation for the key "a" to the previous leader.

Ans:

3. Has the PUT operation been replicated? Indicate which steps lead to a new election and which ones do not. Justify your answer using the statuses returned by the servers.

Ans:

4. Shut down two servers: first shut down a server that is not the leader, then shut down the leader. Report the status changes of the remaining server and explain what happened.

Ans:

5. Can you perform GET, PUT, or APPEND operations in this system state? Justify your answer.

Ans:

6. Restart the servers and note down the changes in status. Describe what happened.

Ans:

## Network Partition

For the first experiment, create a network partition with 2 servers (including the leader)
on the first partition and the 3 other servers on the other one. Indicate the changes that occur in the status of a server on the first partition and a server on the second partition. Reconnect the partitions and indicate what happens. What are the similarities and differences between the implementation of Raft used by your key/value service (based on the PySyncObj library) and the one shown in the Secret Lives of Data illustration from Task 1? How do you justify the differences?

Ans:

For the second experiment, create a network partition with 3 servers (including the
leader) on the first partition and the 2 other servers on the other one. Indicate the changes that occur in the status of a server on the first partition and a server on the second partition. Reconnect the partitions and indicate what happens. How does the implementation of Raft used by your key/value service (based on the PySyncObj library) compare to the one shown in the Secret Lives of Data illustration from Task 1?

Ans: 


# Task 4

1. Raft uses a leader election algorithm based on randomized timeouts and majority voting, but other leader election algorithms exist. One of them is the bully algorithm, which is described in Section 5.4.1 of the Distributed Systems book by van Steen and Tanenbaum. Imagine you update the PySyncObject library to use the bully algorithm for Raft (as described in the Distributed Systems book) instead of randomized timeouts and majority voting. What would happen in the first network partition from Task 3?

Ans: 

2. Why is it that Raft cannot handle Byzantine failure? Explain in your own words.

Ans: 
