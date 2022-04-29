---
layout: post
title: "Visual simulation of consistent hashing"
date: 2021-09-22 06:02:44 +0530
comments: true
categories: system-design caching simufast
image: /images/visual-simulation-of-consistent-hashing.gif
---
Caching is an important aspect of the high performance applications. As the data volume increases, the cached data needs to be distributed across multiple servers. We need to make sure the following objectives are met while doing so.

* Maximize the cache hits: This will reduce the load on primary data source and reduce the overall latency.
* Distribute the data and traffic evenly: This ensures optimal use of servers and avoid overloading a subset of the servers.

As the title of this post suggests, we will look into how [consistent hashing](https://en.wikipedia.org/wiki/Consistent_hashing) can be used to achieve the above objectives. Before that, let's look at using a straight forward approach for solving the problem.

<!-- more -->

## Modulo Hashing

In this approach, we hash the requests based on a key and use the formula `hash(key) % number_of_servers` to route the request to appropriate cache server. For example: If a key "apple" hashes to 14 and we have 3 cache servers, the request for "apple" will be forwarded server number 2 as `14 % 3 = 2`.

Let's simulate this for 3 cache servers, 100 unique keys, 300 random requests and see how it performs.

Click the play button below and check the stats. Increase the speed to fast forward.

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/simufast@0.0.6/src/simufast.css">
<script src="https://cdn.jsdelivr.net/npm/simufast@0.0.6/dist/main.js"></script>

<script type="text/javascript">
    simufast.run((player) => {
        const moduloHash = new simufast.routing.ModuloHash(player);
        const simulation = new simufast.cache.MultiNodeCacheSimulation(moduloHash, {
            nodes: ["S0", "S1", "S2"]
        });
        const keys = simufast.utils.randStringArray(100);
        const commands = [];
        for (let i = 1; i <= 300; i++) {
            const key = simufast.utils.getRandomValueFromArray(keys);
            commands.push(() => simulation.getOrFetch(key, () => `${key}'s value from data source`));
        }
        player.experiment({
            name: 'Modulo Hashing: Static nodes',
            drawable: simulation,
            commands: commands
        });
    }, {
        statsExpanded: true
    });
</script>

Let's analyse the results

* Cache Hit Ratio: As expected, 2/3rd of the requests(67%) are returned from the cache. This is good.
* Data and load distribution: The load is not equally distributed across all servers, but it is fairly distributed. It is purely based on the distribution property of the hash function.

Modulo hashing works well for a fixed number of servers. But in many cases, we need to add or remove servers as per the variation in traffic volume. And, servers can crash sometimes. Let's simulate the following dynamic nodes scenario with modulo hashing and see how it performs.

* 3 servers(S0,S1,S2), 100 unique keys, 300 random requests as earlier
* 1 server(S3) is added after the 100th request
* 1 server(S1) is removed after the 200th request

<script type="text/javascript">
    simufast.run((player) => {
        const moduloHash = new simufast.routing.ModuloHash(player, {maxNodes: 4});
        const simulation = new simufast.cache.MultiNodeCacheSimulation(moduloHash, {
            nodes: ["S0", "S1", "S2"]
        });
        const keys = simufast.utils.randStringArray(100);
        const commands = [];
        for (let i = 1; i <= 100; i++) {
            const key = simufast.utils.getRandomValueFromArray(keys);
            commands.push(() => simulation.getOrFetch(key, () => `${key}'s value from data source`));
        }
        commands.push(() => simulation.addNode("S3"));
        for (let i = 1; i <= 100; i++) {
            const key = simufast.utils.getRandomValueFromArray(keys);
            commands.push(() => simulation.getOrFetch(key, () => `${key}'s value from data source`));
        }
        commands.push(() => simulation.removeNode("S1"));
        for (let i = 1; i <= 100; i++) {
            const key = simufast.utils.getRandomValueFromArray(keys);
            commands.push(() => simulation.getOrFetch(key, () => `${key}'s value from data source`));
        }
        player.experiment({
            name: 'Modulo Hashing: Elastic nodes',
            drawable: simulation,
            commands: commands
        });
    }, {
        statsExpanded: true
    });
</script>

Let's analyse the results

* Cache Hit Ratio: This drops from ~67% earlier to ~45%.
* Data and load distribution: The total number of keys across the servers has increased. This indicates some keys are stored in multiple servers.

This is because, many keys are mapped to a different server when the number of servers change. For example: a key "orange" with hash value 11 is initially routed to server `S2` when there are 3 servers(`11 % 3 = 2`), whereas it is routed to server `S3` when there are 4 servers(`11 % 4 = 3`). This leads to ineffective use of cache.

## Consistent Hashing

Consistent Hashing has a different approach to address the drawbacks of the modulo hashing with dynamic nodes. Let's start with the basic concepts of consistent hashing.

* The servers(called as nodes) are hashed and mapped to a number in a fixed range. This range can be imagined as a real number(not an integer) between 0 and 360 to represent it as points on a circle. The node is placed on this circular ring based on its hash value mapped to range 0-360. For example: If hash value of server `S1` maps to 90, it will be placed as point at 90 degrees on the circumference of the circle.
* The keys are hashed similarly and mapped to a point on the circle. The request for this key is routed to the closest node in the clockwise direction (It can be anticlockwise as well, as long as the same direction is used for all the keys).

Let's play this simulation in 0.5x speed and visualise this basic concept.

<script type="text/javascript">
    simufast.run((player) => {
        const consistentHash = new simufast.routing.ConsistentHash(player, {
            nodeReplicationFactor: 1
        });
        const simulation = new simufast.cache.MultiNodeCacheSimulation(consistentHash);
        const keys = simufast.utils.randStringArray(100);
        const commands = [
            () => simulation.addNode("S1"),
            () => simulation.addNode("S2"),
            () => simulation.addNode("S3"),
        ];
        for (let i = 1; i <= 10; i++) {
            const key = simufast.utils.getRandomValueFromArray(keys);
            commands.push(() => simulation.getOrFetch(key, () => `${key}'s value from data source`));
        }
        player.experiment({
            name: 'Consistent Hashing: Basic Concept',
            drawable: simulation,
            commands: commands
        });
    }, {
        speed: 0.5
    });
</script>

Now that we understand the basic concept, let's run the simulation and observe the stats for 3 servers, 100 unique keys, 300 random requests. (Please increase the simulation speed when required to fast forward to the final stats)

<script type="text/javascript">
    simufast.run((player) => {
        const consistentHash = new simufast.routing.ConsistentHash(player, {
            nodeReplicationFactor: 1
        });
        const simulation = new simufast.cache.MultiNodeCacheSimulation(consistentHash, {
            nodes: ["S1", "S2", "S3"],
        });
        const keys = simufast.utils.randStringArray(100);
        const commands = []
        for (let i = 1; i <= 300; i++) {
            const key = simufast.utils.getRandomValueFromArray(keys);
            commands.push(() => simulation.getOrFetch(key, () => `${key}'s value from data source`));
        }
        player.experiment({
            name: 'Consistent Hashing(Basic): Static nodes',
            drawable: simulation,
            commands: commands
        });
    }, {
        speed: 5,
        statsExpanded: true
    });
</script>

We can observe that, cache hit ratio and load distribution is very similar to that of modulo hashing. This is expected as the algorithm behaves almost the same for fixed number of servers.

Let's see how this basic concept of consistent hashing handles the addition and removal of the nodes using the below scenario

* 3 servers(S1,S2,S3), 100 unique keys, 300 random requests as earlier
* 1 server(S4) is added after the 100th request
* 1 server(S1) is removed after the 200th request

<script type="text/javascript">
    simufast.run((player) => {
        const consistentHash = new simufast.routing.ConsistentHash(player, {
            nodeReplicationFactor: 1
        });
        const simulation = new simufast.cache.MultiNodeCacheSimulation(consistentHash, {
            nodes: ["S1", "S2", "S3"]
        });
        const keys = simufast.utils.randStringArray(100);
        const commands = [];
        for (let i = 1; i <= 100; i++) {
            const key = simufast.utils.getRandomValueFromArray(keys);
            commands.push(() => simulation.getOrFetch(key, () => `${key}'s value from data source`));
        }
        commands.push(() => simulation.addNode("S4"));
        for (let i = 1; i <= 100; i++) {
            const key = simufast.utils.getRandomValueFromArray(keys);
            commands.push(() => simulation.getOrFetch(key, () => `${key}'s value from data source`));
        }
        commands.push(() => simulation.removeNode("S1"));
        for (let i = 1; i <= 100; i++) {
            const key = simufast.utils.getRandomValueFromArray(keys);
            commands.push(() => simulation.getOrFetch(key, () => `${key}'s value from data source`));
        }
        player.experiment({
            name: 'Consistent Hashing(Basic): Elastic nodes',
            drawable: simulation,
            commands: commands
        });
    }, {
        speed: 5,
        statsExpanded: true
    });
</script>

Let's analyse the results

* Cache Hit Ratio: The cache hit ratio is ~60% compared to ~50% in modulo hashing. This is because very few keys are remapped when a node is added or removed.
* Data and load distribution: The load distribution is skewed. In the above scenario(with the chosen node names), the new node `S4` doesn't get many requests due to its proximity to node `S3`.

Consistent hashing solves this load distribution problem by placing each node at multiple points on the ring. These points are called as virtual nodes. For example, if we need to represent node `S1` as 4 points on the ring, we place virtual nodes `S1-1` to `S1-4` on the ring using same logic as earlier. This allows multiple small fragments of the ring to be mapped to a single node.

Let's simulate the previous elastic nodes scenario with 12 virtual nodes per node.

<script type="text/javascript">
    simufast.run((player) => {
        const consistentHash = new simufast.routing.ConsistentHash(player, {
            nodeReplicationFactor: 12
        });
        const simulation = new simufast.cache.MultiNodeCacheSimulation(consistentHash);
        const keys = simufast.utils.randStringArray(100);
        const commands = [
            () => simulation.addNode("S1"),
            () => simulation.addNode("S2"),
            () => simulation.addNode("S3"),
        ];
        for (let i = 1; i <= 100; i++) {
            const key = simufast.utils.getRandomValueFromArray(keys);
            commands.push(() => simulation.getOrFetch(key, () => `${key}'s value from data source`));
        }
        commands.push(() => simulation.addNode("S4"));
        for (let i = 1; i <= 100; i++) {
            const key = simufast.utils.getRandomValueFromArray(keys);
            commands.push(() => simulation.getOrFetch(key, () => `${key}'s value from data source`));
        }
        commands.push(() => simulation.removeNode("S1"));
        for (let i = 1; i <= 100; i++) {
            const key = simufast.utils.getRandomValueFromArray(keys);
            commands.push(() => simulation.getOrFetch(key, () => `${key}'s value from data source`));
        }
        player.experiment({
            name: 'Consistent Hashing(Full): Elastic nodes',
            drawable: simulation,
            commands: commands
        });
    }, {
        statsExpanded: true
    });
</script>

Let's analyse the results

* Cache Hit Ratio: The cache hit ratio remains good i.e. ~60% compared to ~50% in modulo hashing.
* Data and load distribution: The load is distributed a lot better now. The new node `S4` gets a fair amount of traffic compared to earlier. This is because node `S4` is mapped multiple fragments of the ring, increasing its chance of fair share of traffic.

If you would like to simulate your own scenarios, please modify this [JSBin code](https://jsbin.com/fuvavun/edit?html,output) and run own experiments.

## Conclusion

Consistent hashing has proven to be a useful technique since its inception in [1997](https://www.cs.princeton.edu/courses/archive/fall09/cos518/papers/chash.pdf) and it is used in many [well known](https://en.wikipedia.org/wiki/Consistent_hashing) distributed systems because of the simplicity and the benefits it offers. The optimization of consistent hashing does not end with what we have read so far. For example: checkout [this blog](https://medium.com/vimeo-engineering-blog/improving-load-balancing-with-a-new-consistent-hashing-algorithm-9f1bd75709ed) or [the video](https://www.youtube.com/watch?v=jk6oiBJxcaA&ab_channel=GoogleTechTalks) by vimeo engineering on their practical usage and adaptation.

> Meta: You can find the the code used for the above simulations [here](https://raw.githubusercontent.com/endeepak/endeepak.github.io/source/source/_posts/2021-09-22-visual-simulation-of-consistent-hashing.markdown).

## Acknowledgements

* The idea of visual simulation was inspired by the amazing interactive posts like [this](https://ciechanow.ski/internal-combustion-engine/) by [@bciechanowski](https://twitter.com/bciechanowski)
