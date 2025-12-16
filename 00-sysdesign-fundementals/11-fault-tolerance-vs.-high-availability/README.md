## Fault Tolerance vs. High Availability


### Fault Tolerance

Fault Tolerance refers to a system's ability to continue operating without interruption when one or more of its components fail.

**Characteristics**

Redundancy: Incorporates redundancy in system components (like servers, networks, storage) to ensure no single point of failure.

Automatic Failover: Automatically switches to a redundant or standby system upon the failure of a system component.

No Data Loss: Ensures that no data is lost in the event of a failure.

### High Availability

High Availability refers to a system's ability to remain operational and accessible for a very high percentage of the time, minimizing downtime as much as possible.

**Characteristics**
Uptime Guarantee: Designed to ensure a high level of operational performance and uptime (often quantified in terms of “nines” – for example, 99.999% availability).

Load Balancing and Redundancy: Achieved through techniques like load balancing, redundant systems, and clustering.

Rapid Recovery: Focuses on quickly restoring service after a failure, though a brief disruption is acceptable.

