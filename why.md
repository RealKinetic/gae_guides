THIS MAY MAKE MORE SENSE AS A GENERAL GAE POST AS A SETUP.

We're big fans of Google App Engine (GAE). It's our favorite go to tool kit to get a project started. Even if it's not a long term fit for the project we still find it is great for the initial prototype. It comes with a full suite of tools from multiple storage solutions including cache to queues to search. There are limitations for sure. But in our experience it has covered the 80% case of nearly every project we've done. It also comes with a SDK for easy local usage to help create a pretty solid local development experience. And best of all it handles the hard parts. It comes with a built in scheduler for spinning up and down instances. The datastore handles the sharding and partitioning of tables to allow near infinite scale with no operational burden. And even supports multi-tenancy with the support of namespaces.

However there are some rough edges. There are some design patterns that are a bit unique to GAE. Although many are more common these days with the emergance of Microservices and serverless. Part of what allows GAE to be a low operational overhead system is it's limitations and boundaries. To get the full value of GAE and not cause scale and performance issues it's important to learn the good design patterns for GAE. As well as learning the gotchas to avoid. We hope to turn this into a series of posts that walk through the good and bad patterns on GAE.




