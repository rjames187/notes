# Introduction to Data-Intensive Applications

It is important to choose the right data systems (building blocks) for an application.

Applications have such wide-ranging requirements that multiple different data systems are often needed. They are stitched together with application code creating a new composite data system hidden behind an API abstraction.

Examples of tricky questions when designing data systems:
- How to ensure data remains correct and complete when things go wrong?
- How to consistently provide good performance to clients?
- How to handle an increase in load?
- How to design a good API?

Examples of factors influencing data system design:
- Skills and experience of people involved
- Legacy system dependencies
- Delivery timeline
- Tolerance for risk
- Regulatory constraints

Three important concerns for data systems:
- Reliability
- Scalability
- Maintainability

## Reliability

Examples of typical reliability expectations:
- Functions in a way the user expects
- Can tolerate user mistakes
- Performance is good for required usecase under expected load and data volume
- Prevents unauthorized access and abuse

A reliable system continues to work correctly in the face of faults (things that go wrong). Fault-tolerant or resilient systems can anticipate faults and cope with them.
It is preferable to tolerate and cure most kinds of faults rather than prevent them. Unlike a fault, a failure is when the whole system stops working correctly.

### Hardware Faults

Examples of hardware faults:
- Hard disk crash
- Faulty RAM
- Power grid blackout
- Unplugged cable

Hard disks have a mean time to failure of 10 to 50 years so it is important to add redundancy to hardware components. 

Examples of hardware component redundancy:
- RAID configuration
- Dual power supplies and hot-swappable CPUs
- Batteries and diesel generators for backup power

Increasing application data volumes and computing demands have led to the need for multi-machine redundancy in addition to hardware component redundancy. Multi-machine redundancy has operational advantages as it allows for rolling upgrades.

### Software Errors

Software errors are harder to anticipate than hardware faults because they are correlated across nodes.

Examples of software faults:
- A crash caused by bad input
- A runaway process exhausting a resource
- A service slows down, becomes unresponsive, or returns corrupted responses
- Cascading failures

Bugs caused by software faults often lie dormant for a while before being triggered by a specific set of circumstances. Software may make an assumption about the environment that eventually stops being true.

Ways to prevent and deal with software faults:
- Consider assumptions and interactions in the system
- Thorough testing
- Process isolation
- Allow processes to crash and restart
- Measure, monitor, and analyze system behavior in production
- The system can check that it upholds guarantees and raise an alert if not

### Human Errors

Humans are unreliable. Configuration errors by operators are a leading cause of outages.

Ways to prevent and deal with human errors:
- Design good abstractions that aren't too restrictive that make it easy to do the right thing and hard to do the wrong thing
- Decouple places where people make lots of mistakes from places where they can cause failures; for example, using sandbox environments
- Test thoroughly with automated (unit and whole system tests) and manual tests
- Allow quick and easy recovery from human errors (fast roll back and gradual roll out)
- Setup detailed monitoring of performance metrics and error rates to detect early warning signals to detect whether assumptions are being violated
- Good management practices and training

### The Importance of Reliability

Even in non-critical applications, developers have a responsibility to users. However, there are situations where it can make sense to sacrifice reliability (prototypes or narrow profit margins).

## Scalability

Scalability is a system's ability to cope with increased load.

### Describing Load

Load parameters are numbers used to describe load. The best choice of load parameters depends on the system's architecture.

Examples of load parameters:
- requests per second
- read to write ratio
- concurrent users
- cache hit rate

Load parameters can be very specific to the system. For Twitter, a key load parameter is the distribution of followers per user weighed by how often the users post.

### Describing Performance

It is important to consider how an increase in a load parameter can affect the performance of a system and much resources need to be increased so that performance reamins unchanged.

Percentiles are a good metric for response times because they show how many users experienced a delay. P95, P99, and P999 response times are tail latencies. A service level objective (SLO) or service level agreement (SLA) might set an expectation for median response time and tail latency.

## Sources
*Designing Data-Intensive Applications* by Martin Kleppmann
