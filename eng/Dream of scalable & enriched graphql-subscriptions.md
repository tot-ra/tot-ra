
![https://miro.medium.com/max/1400/1*Rvb94EOQA-BzsmYJ4TklGg.jpeg](https://miro.medium.com/max/1400/1*Rvb94EOQA-BzsmYJ4TklGg.jpeg)

Stylized photo of Jagala juga (Estonia), original photo by Aleksandr Abrosimov, Wikimedia Commons

[Last time](https://medium.com/pipedrive-engineering/journey-to-federated-graphql-2a6f2eecc6a4), I wrote about 5-year long journey of having GraphQL in Pipedrive. Now, let me tell you about a 10-year long journey of delivering websocket events to frontend.. and maybe help you out with it too.

![https://miro.medium.com/max/1400/1*xffiu2fshUMbBFm-JlSAGw.gif](https://miro.medium.com/max/1400/1*xffiu2fshUMbBFm-JlSAGw.gif)

# **Why**

The **product need** of asynchronous events comes up any time user needs to be notified about something by the server.

- you’ve got an instant message
- uploaded image has finished resizing
- your colleague has changed an avatar
- your bulk change of 10k deals has progressed to 99%

So, async events **enrich UX with details & interactivity**

But most importantly, they solve a problem of having **inconsistent data** displayed or stored on the browser side. Without such updates, other user won’t see _Pipedrive deals renames_; another browser tab won’t receive _activities deleted_. Even in the same browser tab, your views may not talk well with each other, lacking single storage and be out-of sync.

# **Historical dive**

Luckily, Pipedrive has “solved” this issue 10 years ago. In 2012 [Andris](https://www.linkedin.com/in/andris-reinman/), [Kapp](https://www.linkedin.com/in/martinkapp/) & [Tajur](https://www.linkedin.com/in/martintajur/) have developed a **socketqueue** service, which to this day transports API events to the front-end leveraging [sockjs](https://www.npmjs.com/package/sockjs) library.

Tajur’s talk in 2016 was one of the reasons I’ve re-created similar setup, got puzzled how I should manage user connections with [socket.io](https://socket.io/) & scale it beyond one server and ultimately joined Pipedrive to find out. I was fascinated by event streaming and the possibilities it brings.

As you can see, his talk is more about [RabbitMQ](https://www.rabbitmq.com/) — a message broker that stands between php monolith and socketqueue.

# **Pros and cons**

In-memory queues have allowed Pipedrive to withstand bursts of events, triggered by external integrations or in-house things like deal import from xls file or a bulk edit.

Api event that is pushed to the RabbitMQ and to the frontend is **denormalized**, having different entities. That’s good, because you have a **consistent snapshot** of multiple things at the same time and you recipients **don’t need to re-query** anything…

But it is also hugely **inefficient**, as all browser tabs receive all possible changes, wether they want it or not, sometimes having reached 80GB per day of traffic for a single customer.. while web app on his laptop CPU is doing all of the heavy filtering.

So we can’t keep infinitely bloating event schema, making it even heavier.

![https://miro.medium.com/max/1400/1*SEK3NXeBwKWHvw4fgcbVBg.png](https://miro.medium.com/max/1400/1*SEK3NXeBwKWHvw4fgcbVBg.png)

Cell logic as number in a web-socket URL

Second problem is a **noisy neighbour**. Scalability, as I mentioned, is solved with “cell logic” (in a good scenario — aka tenant isolation) which means sharding socketqueue application servers by company ID. This also means that we must have **fixed amount of containers** and fixed amount of queues to ensure predictable routing.

So problem arises whenever you have one company generating thousands of events — they are not distributed among all servers, instead they **spike CPU** of a single node to the point of health check failure. There isn’t any room to vertically grow single-core CPU, only to double amount of servers which means **wasted infrastructure resources**.

![https://miro.medium.com/max/1400/1*2pcMt6Bjn_pgBLO9CLjjOA.png](https://miro.medium.com/max/1400/1*2pcMt6Bjn_pgBLO9CLjjOA.png)

noisy neighbour CPU spike of particular pod

Third issue is **visibility & tracing**. Same as with REST API, without proper documentation (wether it is swagger or type definitions), it is very hard to understand what can be inside of the event schema and most important, who uses what in case you want to do a breaking change. Which leads to a vicious cycle of nobody removing anything from the event & bloating it even more.

Finally, if web-socket connection is down because user went into a tunnel or just closed his laptop — he may loose events. Could that cause a loss of _a deal_ and profits for the user?

Over the years we’ve optimized the hell out of socketqueue, so its

- multi-threaded as it listens queues and WS connection handling
- compresses traffic
- checks permissions & visibility

but what if we could do better..? 🤔

# **How**

Idea behind [graphql subscriptions](https://spec.graphql.org/June2018/#sec-Subscription-Operation-Definitions) is pretty simple — you declare only what you want to receive, given some filtering arguments. And it will be server’s job to intelligently filter events.

I can’t do a better job of explaining basic setup than [Ben](https://twitter.com/benawad) here 😁. So what he shows here is a single server instance where pubsub is just in-memory event routing directly from mutation. But in our case there are no mutations — real changes are generated by the php legacy far-far away.

# **Mission scope**

So what I wanted to achieve was

- demo graphql subscriptions in production, having explicit schema
- test horizontal pod scaling
- increase reliability by using kafka instead of rabbitMq
- use small, normalized, domain events to keep things simple
- keep event delivery latency below 2 sec [to not be slower than socketqueue we’re trying to replace]

## **Risks**

What I didn’t know..

- how do we authenticate?
- what protocol should we use? SSE? WS? whats this [mercure](https://github.com/dunglas/mercure) thing.. and does some of it fallback to pooling?
- will WS/TCP connections be bound to same pod or not?
- can we do multiple subscriptions at a time?

![https://miro.medium.com/max/3920/1*K-aB3FKXH0jXeG4HiLRdVw.png](https://miro.medium.com/max/3920/1*K-aB3FKXH0jXeG4HiLRdVw.png)

![https://miro.medium.com/max/4004/1*5q23Z56at20SsvRkOB_6Hw.png](https://miro.medium.com/max/4004/1*5q23Z56at20SsvRkOB_6Hw.png)

- what storage / transfer medium do we use? DB? redis streams? kafka? redis pubsub? KTable?
- can we rewind events in time in case user got disconnected? would we need to store cursor ID per entity? also, how does live query work?
- how do we filter events by company, user, session, entity? is there a standard for subscription filter fields?
- can we re-use federated graphql schema? can we enrich events? what should be QoS / retry logic if gateway is down? Lee Byron went into it at depth [in his talk over 5 years ago](https://youtu.be/rapO30fpREg?t=6675).
- how many items can we subscribe to?
- how do we unsubscribe? or disconnect users when they log out?

# **Proof of concept**

Before the mission, I made a simple node service that connected to kafka and proxied events without any filtering. This was already promising. But for a mission I wanted to try out [_**go**_](https://go.dev/) to have all CPUs to be involved to maximize that efficiency, while this PoC was a fallback (that proved itself very useful)

![https://miro.medium.com/max/2256/1*8i540taseWXr_TAV2ysBZw.png](https://miro.medium.com/max/2256/1*8i540taseWXr_TAV2ysBZw.png)

![https://miro.medium.com/max/4952/1*z39G_eYPNR7Zuu_Hl1dK7g.png](https://miro.medium.com/max/4952/1*z39G_eYPNR7Zuu_Hl1dK7g.png)

# **Liftoff**

Four of us ([myself](https://www.linkedin.com/in/kurapov/), [Pavel](https://www.linkedin.com/in/pavel-nikolajev-84184a64/), [Kristjan](https://www.linkedin.com/in/kristjan-luik-89bb03122/) and [Hiro](https://www.linkedin.com/in/abhishek-goswami-591541b1/)) have set off for entire summer to try and do just that.. explore the unknown 🛸 and bring value back to the launchpad.

![https://miro.medium.com/max/1400/1*QlBXCk8EjRtp5n_Fxr2xTw.png](https://miro.medium.com/max/1400/1*QlBXCk8EjRtp5n_Fxr2xTw.png)

## **gophers**

We looked into [graph-gophers/graphql-go](https://github.com/graph-gophers/graphql-go) as first basis and within couple of days we’ve re-created PoC of subscribing to kafka events

- We’ve hit the dilemma of GraphQL integer spec [not matching go’s int32](https://github.com/graphql/graphql-js/issues/292#issuecomment-186702912).
- Got per-property filtering and per-argument filtering working
- Hit need of SSL for WSS to work, otherwise CORS blocked requests
- Got [nginx to proxy web socket requests](http://nginx.org/en/docs/http/websocket.html)!
- We found that there are two transport protocols, as websocket is a loose tranport so any library can implement whatever it wants. [Gophers implemented](https://github.com/graph-gophers/graphql-transport-ws) older version, that was compliant with apollo & graphiql but not with graphql-ws
- Hit issue that apollo federation lib did not like `Subscription` in [schema registration](https://github.com/pipedrive/graphql-schema-registry). We figured that we can clean only Subscription type from root to have other types checked for collisions

Basic filtering was pretty simple, you just need to connect two streams (channels) — kafka and web sockets (that gophers need as schema resolvers) while having a goroutine in the middle transforming data

![https://miro.medium.com/max/3712/1*zXxbPFcX5Xqu7zGz8k3kSw.png](https://miro.medium.com/max/3712/1*zXxbPFcX5Xqu7zGz8k3kSw.png)

![https://miro.medium.com/max/3668/1*0dXQgXJcVTET0AswerSujA.png](https://miro.medium.com/max/3668/1*0dXQgXJcVTET0AswerSujA.png)

We’ve hit a major [roadblock with gophers](https://github.com/graph-gophers/graphql-transport-ws/pull/9) and JWT authentication, we’re not that experienced in _go_ & don’t have much time to start changing a major framework.

We also figured that enrichment is not a priority, and we need to bring code into production 🚀 and test scaling. Thats the agile way.

Second big blocker — entity visibility checks. Reading domain events from kafka means that we don’t have extra information, like who can see a deal. We need to query some service about this. Doing it on every event of every company likely can lead to DoS 🤦‍♂️

## **Gqlgen**

We scratch gophers, switch to gqlgen. Channel exchange logic remains the same and we get JWT token authentication working. Difference is that now subscription events are autogenerated based on schema.graphql.

Next, I find that exchange (in-memory pub-sub in go) is kinda flawed, we need at least 1 subscription for streaming to take place.. newer version is a bit better and allows to have channel per-company.

![https://miro.medium.com/max/2532/1*_tIlshM1g5F8tYgIN4qAfQ.png](https://miro.medium.com/max/2532/1*_tIlshM1g5F8tYgIN4qAfQ.png)

![https://miro.medium.com/max/2176/1*w6B9z69_upq05WXXb2Vt5Q.png](https://miro.medium.com/max/2176/1*w6B9z69_upq05WXXb2Vt5Q.png)

We also start brainstorming how schema should look like to have it scalable across different entitites, have filters, have multiple entity actions and multiple ids supported.

1 month in. We have UI working with hacked but performant permission check. We take vacations to cool off 🌴 And only now I see this 🤯 presentation by [Mandi Wise](https://twitter.com/mandiwise) from Apollo, even though I’ve gone through most of youtube videos by that time..

## **Final architecture**

So we come back and scratch gqlgen again, it was painful but well.. agile. We split service in **two**, adding redis pub-sub with replication and sentinel in between

![https://miro.medium.com/max/1400/1*JDm6Oo8YfE1-r7lzttAyNQ.png](https://miro.medium.com/max/1400/1*JDm6Oo8YfE1-r7lzttAyNQ.png)

- graphql-subscription-workers remains in go, but becomes way simpler — it is be able to scale well to consume all kafka partitions. Workers filter out events by doing 10x parallel permissions checks and transform event schema if needed
- graphql-subscriptions deals with connections & scales horizontally as much as needed. It leverages great [graphql-ws](https://github.com/enisdenjo/graphql-ws) library that has all sorts of hooks, so kudos to [Denis Badurina](https://github.com/enisdenjo)
- redis pub-sub serves as exchange broker doing most of the filtering & sharding by company, user and entity ids.

For example, worker pushes to message channel: <companyId>.deal.<dealId>. This is very similar to rabbitMQ routing as it also provides regex wildcard matching by the subscribers. This allows us to theoretically subscribe to <companyId>.* and handle all events if such need arises.

## **Subscription schema types**

So as you can see from the diagram, we followed Mandi’s advice and wrote event enrichment by querying gateway. This did require polling of federated schema and hard-coded resolvers of which query specific entity must make, but it gives immence flexibility to the frontend

![https://miro.medium.com/max/2376/1*zBOSyHn9wocWYbXR7r80gA.gif](https://miro.medium.com/max/2376/1*zBOSyHn9wocWYbXR7r80gA.gif)

![https://miro.medium.com/max/3504/1*dauu7v8ZBwU667SdCCBBnA.png](https://miro.medium.com/max/3504/1*dauu7v8ZBwU667SdCCBBnA.png)

Schema choices we had to make:

- we do not merge original event with enriched one, although we could. This is because we don’t have guarantees that graphql-service responds in timely manner. Kafka “raw” event data is much more reliable. We also have minor differences in schema when referencing is used (raw deal.stageId vs enriched [deal.stage.id](http://deal.stage.id))
- we added “delta” as JSON, mostly because frontend really wanted to use that in case frontend storage wanted to update only specific keys. This requires specific shape of kafka event though
- we have a generic event type (dealEvent) along side with action-specific events (dealAdded), because generic events take into account correct order of events for the same entity ( added -> changed -> deleted ) which is not guaranteed chronologically if you subscribe to separate types

## **Live queries**

This a cool concept that elegantly fixes a problem which I was really fixated on — connection loss.

From to [Laurin’s article](https://the-guild.dev/blog/subscriptions-and-live-queries-real-time-with-graphql) by The Guild, we’ve got same **fetchOrSubscribe** function which first does enrichment request and then subscribes to kafka events using async iterator magic. The difference however is in schema — we use “liveQuery” as an argument, not as a root property.

This thing alone meets 99% of consistency requirements in web apps, without the need to rewind events (using [redis streams](https://redis.io/docs/data-types/streams/)?). So “cursor-based query revalidation and diffing” that [Ben Newman suggested at last Graphql Summit](https://twitter.com/benjamn/status/1579891243798401026) seems like a very rare use case where you want to rewind events as a niche product feature (in gaming?)

# **Performance & security testing**

Finally, by the mission end we tested how well our solution scales in production, gradually rolling it out to real customers.

![https://miro.medium.com/max/1400/1*mHE_vMsNTPr25y7IkePG2Q.png](https://miro.medium.com/max/1400/1*mHE_vMsNTPr25y7IkePG2Q.png)

Performance testing in production

This did reveal interesting things - if you subscribe to _deals_ by IDs… and if you do this in _pipeline view_ where user can scroll very fast, you can have waves of subscriptions. So better to avoid blocking permission checks onSubscribe and have some throttling logic on frontend.

We have had also observed random pods suddenly loosing all of their connections, that has lead us to revise memory limits & [max_old_space_size bug](https://github.com/nodejs/node/issues/35573).

We have implemented various security limitations on amount of connections, subscriptions, timeouts, taking into account logouts, permission changes etc. Not only because of external risks, but also because we did find that Redis needs to do a lot of CPU to match **pub**lishers to **sub**scribers.

# **Future steps**

Theoretically, we could shard Redis instances by entity types if we would face performance problems.

We could force socketqueue deprecation by _temporarily_ adding api events as a universal subscription, proxying all data to the frontend without any filtering. Sacrifice efficiency for the sake of having single transport layer.

We have had service operational for over a year now and so far got ~10 entities supported, tied to different kafka topics

## **In defence of GraphQL**

Adoption of GraphQL and subscriptions in particular is seen as slow, although it is voluntary.

With [codegen](https://youtu.be/UZWe5Usun7I?t=4130) that can wrap REST API, it can be seen as complex, without — as too demanding of service refactoring, alongside a REST API — as redundant, with [federation](https://www.apollographql.com/docs/federation/) — as having too many layers, with [dataloader](https://github.com/graphql/dataloader) but without observability — as doing [too many poorly batched RPC calls](https://twitter.com/Popeska/status/1592177670212943873).

Return on investment is not seen, if _value_ is not measured.

GraphQL has taken onto itself a big responsibility of making web more transparent, predictable, reusable and more efficient. I urge developers 🌞 to have professional patience — educate your colleagues, deliver **hard metrics** to managers, proving GraphQL superiority.

My dream is that there will come a day when devs will replace schema-less WS events & REST APIs with GraphQL and subscriptions for anyone to consume. Clients are our future, this is the way