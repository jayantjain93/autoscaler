## Debugging Snapshot for Cluster Autoscaler

With the growing number of large, autoscaled clusters we are increasingly dealing with complex customer cases that are very hard to debug. One major difficulty is that we log information about what decision Cluster Autoscaler (CA) took as well as the results of various intermediate steps that lead to this decision, but we don't log the data this decision was based on.

The reason for this is that CA is internally simulating the behavior of the entire cluster and to fully understand any given decision we need to know the exact state of relevant nodes and pods. The volume of logging required to capture all that data would be prohibitive.

Due to the above we're often unable to root cause an issue after the fact and we have to rely on customers keeping their clusters in bad state or issue reoccurring to be able to debug. Even with access to a cluster with an ongoing issue, not all important data can be retrieved.

Collaborators: @JayantJain93 @MaciekPytel 

## Proposal

This document proposed introducing a new "snapshot" feature to Cluster Autoscaler. This feature would allow an engineer debugging an autoscaler issue to manually trigger CA to dump it's internal state.

**This would serve two purposes:**
- An engineer trying to root cause the issue could create a snapshot to access parts of internal CA state that are otherwise not available (ex. node templates used in scale-up simulations).
- An oncall or support engineer could capture the snapshot before applying any mitigation to increase the chances of root causing the issue later on.

We want to capture the data that Cluster Autoscaler is internally using in its scheduler simulations in order to understand the results in those simulations. In particular we want to capture the following:
- **All nodes in Cluster as observed by CA.** This includes all the parts of node spec and status that are relevant for scheduling, as well as the list of pods currently running on that node according to CA.
- **Pod assignment in FilterOutSchedulable phase.** In this step CA is binpacking pending pods on existing nodes to see if scale-up is needed or if some or all of the pending pods can already be scheduled. Many of the most complex issues we've debugged involve CA disagreeing with scheduler in this step. Knowing exactly where CA thinks the pending pods can be scheduled can be very helpful.

**--------------- TODO:  Add more functional requirement as we finalise them internally ---------------**

