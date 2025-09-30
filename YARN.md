# YARN

-   https://www.youtube.com/watch?v=5vmP1-6xd6Y&t=101s

YARN stands for Yet Another Resource Negotiator. It’s the cluster operating system of Hadoop: a general-purpose resource manager + scheduler that decides which app gets how many CPUs, how much memory (and GPUs, disks, etc.) on which machines, and when.

Think of a big conference center:

-   ResourceManager (RM) = the front desk + room booking system for the entire center.
-   NodeManager (NM) = the floor manager on each floor who prepares rooms and reports status.
-   ApplicationMaster (AM) = the organizer of one specific event (e.g., “Spark job” or “Hive query”).
-   Container = a room booking (e.g., “2 hours in Room 3 with 4 tables and a projector”)—i.e., a bundle of resources (memory, vcores, optionally GPUs).
-   Tasks = the sessions that run inside the booked rooms.
