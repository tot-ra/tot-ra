![https://miro.medium.com/max/1400/1*6ADAF1r2LFunXgdklnMRzw.png](https://miro.medium.com/max/1400/1*6ADAF1r2LFunXgdklnMRzw.png)

Image taken from [dgraph.io](http://dgraph.io)

Engineers tend to love good stories, so hopefully our 5-year journey of moving towards API composition with [GraphQL](https://graphql.org/) now in production (serving at peak 110 requests per second at 100ms latency) provides a good story.

[If you’re in a hurry, scroll down to _Lessons learned_ and check out the open-sourced [graphql-schema-registry](https://github.com/pipedrive/graphql-schema-registry).]

![https://miro.medium.com/max/1400/1*d32xg4FUICDlOm9q0PCLkw.png](https://miro.medium.com/max/1400/1*d32xg4FUICDlOm9q0PCLkw.png)

schema registry with some [example schema](https://github.com/jeffwillette/graphql-go-pets-example)

# **Our Requirement**

For years, Pipedrive (which hit 10 years near the beginning of 2020) has had a [public REST API](https://developers.pipedrive.com/docs/api/v1/), as well as hidden, undocumented endpoints for our webapp — one of which is _/users/self,_ that was initially for loading user information but over time became a page-load API, composed of 30 different entity types. It originated in our PHP monolith which by nature is synced. We [tried to split it](https://medium.com/pipedrive-engineering/how-two-developers-accelerated-php-monolith-in-pipedrive-df8a18bc2d8a) into parallel threads but that didn’t work out so well.

![https://miro.medium.com/max/1400/1*pqnU6EvMYnIQmVnnt5V1mA.png](https://miro.medium.com/max/1400/1*pqnU6EvMYnIQmVnnt5V1mA.png)

/users/self latency distribution currently for the remaining traffic

From a maintenance point of view, with every new change, it became messier as nobody wanted to own this enormous endpoint.

# **Direct DB Access PoC**

Let's go back in time to when our devs experimented with graphql.

About 3–4 years ago in the [marketplace](https://marketplace.pipedrive.com/) team, I began hearing new terms like “[elixir](https://elixir-lang.org/)” and “graphql” from [Pavel](https://twitter.com/bashmach), our fullstack engineer. He was involved in a proof-of-concept project which directly accessed MySQL and exposed the _/graphql_ endpoint for querying core Pipedrive entities.

It worked fine in dev, but it wasn’t scalable because our backend was not only CRUD and nobody wanted to rewrite the monolith.

# **Stitching PoC**

Fast forward to 2019 when I saw another internal PoC by a colleague that used GraphQL schema stitching with [graphql-compose](https://github.com/graphql-compose/graphql-compose) and makes requests to our REST API. As you can imagine, this was a major improvement since we wouldn’t need to re-implement all of the business logic and it was just a wrapper.

The downsides in this PoC were:

- **Performance**
    
    . It didn’t have
    
    [dataloader](https://github.com/graphql/dataloader)
    
    , so it had a potential N+1 API call problem. It didn’t limit query complexity and it didn’t have any intermediate caching. On average, the latency was higher than with the monolith.
    
- **Schema management.** With schema stitching, we need to define a schema in the single repo, separate from an actual service that serves data. This complicates deployment, as we would need intermediate backward-compatible deployments to avoid crashes in case service changes.
    

# **Preparation**

In October 2019, I started to prepare for the mission that would move the previous PoC into production, but with a new Apollo federation that [came out](https://www.apollographql.com/blog/apollo-federation-f260cf525d21/) the same year. This would also land into a core team that would maintain the service through the long-term.

## **Gathering developer expectations**

Internally, some developers were skeptical and suggested building an in-house API composition by stitching REST API URLs and their payloads into a single POST request and relying on an internal gateway to split requests on the backend.

Some saw graphql as still too raw to adopt in production & keep a status-quo. Some suggested exploring alternatives, like Protobuf or [Thrift](https://thrift.apache.org/), and using transport conventions like [GRPC](https://grpc.io/), [OData](https://www.odata.org/).

Conversely, some teams went full throttle and already had graphql in production for individual services (insights, teams), but couldn’t reuse other schemas (like User entity). Some ([leads](https://www.pipedrive.com/en/jobs/czech-republic)) used a typescript + [relay](https://relay.dev/) which still needed to figure out [how to federate](https://github.com/apollographql/apollo-server/issues/3159).

Local conference talk about a similar topic (VP of Tech @ [Monese](https://medium.com/@MyMonese))

Researching new tech was exciting:Strict, self-describing API for frontend developers? Global entity declaration and ownership that forces the reduction of duplication and increases transparency? A gateway that could automatically join data from different services without over-fetching? Wow.

I knew that we needed schema management as a service to not rely on a hard-coded one and have visibility on what’s happening. Something like [Confluent’s schema-registry](https://docs.confluent.io/current/schema-registry/index.html) or [Atlassian’s Braid](https://bitbucket.org/atlassian/graphql-braid/src/master/), but not Kafka-specific or written in Java, which we didn’t want to maintain.

# **The plan**

I pitched an engineering mission that focused on 3 goals:

- Reducing initial page load time (pipeline view) by 15%. Achievable by joining some REST API calls into a single /graphql request
- Reducing API traffic by 30%. Achievable by moving deal loading for the pipeline view to graphql and requesting fewer properties.
- Using strict schema in API (for the frontend to write less [defensive code](https://en.wikipedia.org/wiki/Defensive_programming))

I was lucky to get [3 awesome experienced developers](https://github.com/pipedrive/graphql-schema-registry#honorable-mentions) to join the mission, including a PoC author.

Multiple REST API calls from webapp done at different time

![https://miro.medium.com/max/5004/1*jmyHq5UWstzZeCuXSPYHIw.png](https://miro.medium.com/max/5004/1*jmyHq5UWstzZeCuXSPYHIw.png)

![https://miro.medium.com/max/5000/1*_L6W6bxg5jt4oC_Yd6jkAw.png](https://miro.medium.com/max/5000/1*_L6W6bxg5jt4oC_Yd6jkAw.png)

The original plan for the services looked like this:

![https://miro.medium.com/max/1400/1*hZPmw66Kk3e9TtYj-FWNEg.png](https://miro.medium.com/max/1400/1*hZPmw66Kk3e9TtYj-FWNEg.png)

Services we would need to work on

The schema-registry here would be a generic service that could store any type of schema you throw at it as an input ([swagger](https://editor.swagger.io/), typescript, graphql, avro, proto). It would also be smart enough to convert an entity to whatever format you want for the output. The gateway would poll schema and call services that own it. Frontend components would need to download schema and use it for making queries.

In reality however, we implemented only graphql because federation only need this and we ran out of time pretty quickly.

# **Results**

The main goal of replacing messy _/users/self_ endpoint in the webapp was done within the first 2 weeks of the mission 😳 (yay!). Polishing it so that it’s performant and reliable, took most of the mission time though.

By the end of the mission (in February 2020), we _did_ achieve **13%** initial page load time reduction and **25%** on page reload (due to introduced caching) based on the synthetic [Datadog](https://www.datadoghq.com/) test that we used.

We did not reach the goal of traffic volume reduction, because we didn’t reach the refactoring _pipeline_ view in webapp — we still use REST there.

To increase adoption we added internal tooling to ease federation process and recorded onboarding videos for teams to understand how it works now. After the mission ended, IOS and Android clients also migrated to graphql and the teams gave positive feedback.

# **Lessons learned**

Looking back at the mission log that I kept for all of the 60 days, I can outline the biggest issues so that you don’t make the same mistakes

## **Manage your schema**

> Could we have built this ourselves? Maybe, but it wouldn’t be as polished.Mark Stuart, Paypal Engineering

Through the first couple of days, I tried [Apollo studio](https://studio.apollographql.com/) & its [tooling](https://www.apollographql.com/docs/studio/schema-checks/#the-check-process) for CLI to validate schema. The service is excellent and works out-of-the-box with minimal gateway configuration.

Our internal tools to see schema diffs and validate pushed schema from the terminal, similar to Apollo’s tooling

![https://miro.medium.com/max/2612/1*peOiXrxaxYTeJ0kAjXOqnA.png](https://miro.medium.com/max/2612/1*peOiXrxaxYTeJ0kAjXOqnA.png)

![https://miro.medium.com/max/3700/1*0PkQISCe0ux2nG6c6l5wXg.png](https://miro.medium.com/max/3700/1*0PkQISCe0ux2nG6c6l5wXg.png)

As well as it works, I still felt that tying core backend traffic to an external SaaS is too risky for business continuity, regardless of its great features or pricing plans. This is why we wrote a service of our own, with basic functionality — now it’s an open-source [graphql-schema-registry](https://github.com/pipedrive/graphql-schema-registry).

The second reason to have in-house schema-registry was to follow Pipedrive’s [distributed datacenter model](https://medium.com/pipedrive-engineering/tanker-the-story-of-multi-dc-customer-data-migration-framework-ad842c2c6b9c). We don’t rely on centralized infrastructure, every datacenter is self-sufficient. This gives us higher reliability as well as a potential advantage in case we need to open a new DC in places like China, Russia, Iran, or Mars 🚀

## **Version your schema**

Federated schema and graphql gateway are very fragile. If you have type name collision or invalid reference in one of the services and serve it to the gateway, it won’t like it.

By default, a gateway’s behavior is to [poll services for their schemas](https://www.apollographql.com/docs/apollo-server/federation/introduction/#gateway-example) so it's easy for one service to crash the traffic. Apollo studio solves it by validating schema on-push and rejecting registration if it causes possible conflict.

The validation concept is the right way to go, but the implementation also means that Apollo studio is a **stateful** service that holds **current** valid schema. It makes the schema [registration protocol](https://www.apollographql.com/docs/studio/schema/schema-reporting-protocol/#protocol-sequence) more complex and [dependent on time](https://www.apollographql.com/docs/studio/schema/schema-reporting/#rolling-deploys-with-schema-reporting), which can be somewhat hard to debug in case of _rolling deploys_.

In contrast, we tied the service version (based on **docker image hash**) to its schema. Service registers schema also in runtime, but we do it once, without [constant pushing](https://www.apollographql.com/docs/studio/schema/schema-reporting/#rolling-deploys-with-schema-reporting). The gateway takes federated services from service discovery ([consul](https://www.consul.io/)) and asks schema-registry to _/schema/compose_, providing their services’ **versions**.

If schema-registry sees that the provided set of versions is not stable, it falls back to the last registered versions (which are used for schema validation on commit, so should be stable).

![https://miro.medium.com/max/1400/1*Sw5SVUCfE0pU3eqpcVg7EQ.png](https://miro.medium.com/max/1400/1*Sw5SVUCfE0pU3eqpcVg7EQ.png)

![https://miro.medium.com/max/1400/1*ycJcR95HBuDhgkyWkUN2aw.png](https://miro.medium.com/max/1400/1*ycJcR95HBuDhgkyWkUN2aw.png)

Runtime schema registration example using an internal library

Services can serve both REST and Graphql APIs, so we resort to alerts in case schema registration fails, keeping service operational for REST to work.

## **Defining schema based on existing REST is not so straightforward**

Since I didn’t know how to convert our REST API to graphql, I tried [openapi-to-graphql](https://github.com/IBM/openapi-to-graphql), but our [API reference](https://developers.pipedrive.com/docs/api/v1/) didn’t have sufficiently detailed swagger documentation to cover all inputs/outputs at the time.

Asking each team to define schema would take so much time that we just defined the schema for main entities ourselves 😓 based on REST API responses.

Doing this came back to haunt us later, when it turned out that some of the REST API depended on the client that makes requests **OR** had different response format depending on some business logic/state.

For example, **custom fields** affect API response depending on the customer. If you add a custom field to a deal, it will be served as a _hash_ on the same level as _deal.pipeline_id_. Dynamic schema is not possible with graphql federation, so we had to work-around that by moving custom fields into a separate property.

![https://miro.medium.com/max/3040/1*wRvmLci_zeX8SI3XLablcA.png](https://miro.medium.com/max/3040/1*wRvmLci_zeX8SI3XLablcA.png)

![https://miro.medium.com/max/5452/1*jYUN6yru6BRPltGcZvQzqQ.png](https://miro.medium.com/max/5452/1*jYUN6yru6BRPltGcZvQzqQ.png)

Another long-term issue is **naming conventions**. We wanted to use _camelCase_, but since most of REST used _snake_case_, we ended up with a mix for now.

Current Pipedrive’s federated graph (on the left) with 2 federated microservices (out of 539) and not-yet federated leads (on the right), generated with [voyager](https://github.com/APIs-guru/graphql-voyager)

![https://miro.medium.com/max/2420/1*4FDJvhR6Z9zQSCF4_lu-Qg.png](https://miro.medium.com/max/2420/1*4FDJvhR6Z9zQSCF4_lu-Qg.png)

![https://miro.medium.com/max/3860/1*x2jV67R_69ZYy6y75TsDiA.png](https://miro.medium.com/max/3860/1*x2jV67R_69ZYy6y75TsDiA.png)

## **CQRS and cache**

The Pipedrive data model isn’t simple enough to rely only on a TTL-cache.

For example, if a support engineer creates a _global message_ about maintenance from our backoffice, he also expects it to be immediately shown to customers. Those global messages can be shown to _all customers_ or can affect _specific users_ or _companies_. Breaking such cache needs 3 layers.

To handle php monolith with graphql in async mode, we have created a nodejs service (called _monograph_) which caches everything php responded into memcached. This cache has to be cleaned from PHP depending on business-logic making it a bit of an anti-pattern of tightly-coupled cross-service cache.

![https://miro.medium.com/max/1400/1*MWhY3MEiDCMs2lnpWjyvIQ.png](https://miro.medium.com/max/1400/1*MWhY3MEiDCMs2lnpWjyvIQ.png)

You can see the [CQRS pattern](https://martinfowler.com/bliki/CQRS.html) here. Such cache makes it possible to speed-up 80% of requests and get avg latency to the same as php-app, while still having strict schema and no overfetching

Average latency in php-app (left) vs graphql gateway (right) in US region via NewRelic

![https://miro.medium.com/max/3192/1*-6FCWqp-bdqK64_5wq-wtw.png](https://miro.medium.com/max/3192/1*-6FCWqp-bdqK64_5wq-wtw.png)

![https://miro.medium.com/max/3128/1*F6ArQsPeRYYiAeD4zOAOCg.png](https://miro.medium.com/max/3128/1*F6ArQsPeRYYiAeD4zOAOCg.png)

Another complication is the _customer’s language_. Changing this affects so many different entities — from activity types to google map views and to make things worse, the user language is not managed by php-app anymore, its in _identity_ service — and I didn’t want to couple even more services to a single cache.

_Identity_ owns user information, so it emits a change event that _monograph_ listens to and clears its caches. This _means_ that there is some delay (maybe max ~1 sec) between changing language and getting caches cleared, but it's not critical, because a customer won’t navigate away from a page _so fast_ to notice the old language still in the cache.

## **Track performance**

Performance was the main goal and to reach that goal we had to master APM and distributed tracing between microservices to see what is the slowest area. We used Datadog during this time and it showed the main issues.

We also used memcached to cache all 30 parallel requests. The problem (shown on this picture) displays that purple requests to memcached for some of the resolvers throttled up to 220ms, whereas the first 20 requests saved data in 10ms. This was because I used the same [mcrouter](https://github.com/facebook/mcrouter) host for all of the requests. Rotating hosts reduced caching write latency to 20 ms max.

To reduce latency due to network traffic, we got data from memcached with a single batch request for different resolvers using [getMulti](https://github.com/3rd-Eden/memcached#public-methods) and a 5 ms debounce.

Issues during mission — memcached (left), slow single resolver (right) in Datadog distributed tracing

![https://miro.medium.com/max/3700/1*ibZbgnevw9dsYg0nVHbo3Q.png](https://miro.medium.com/max/3700/1*ibZbgnevw9dsYg0nVHbo3Q.png)

![https://miro.medium.com/max/2388/1*bB4N6SdLy0kAQvRO5o3aAQ.png](https://miro.medium.com/max/2388/1*bB4N6SdLy0kAQvRO5o3aAQ.png)

Notice also the yellow bars on the right — that’s the graphql **gateway’s tax** on making data strictly typed after all data is resolved. It grows with the amount of data you transfer.

![https://miro.medium.com/max/1400/1*KKchwFbhA817Ok5Eugq7hQ.png](https://miro.medium.com/max/1400/1*KKchwFbhA817Ok5Eugq7hQ.png)

We needed to find the slowest resolvers because the latency of the entire request depends entirely on them. It was pretty infuriating to see 28 out of 30 resolvers reply in 40ms time.. and 2 of the APIs taking 500ms, because they didn’t have any cache.

We had to move some of these endpoints out of the initialization query to create better latency. So we actually make 3–5 separate graphql queries from frontend, depending on when query is requested (also with some debounce logic).

## **Do not track performance (in production)**

The counter-intuitive clickbait heading here actually means that you should avoid using APMs in production for graphql gateway or apollo server’s built-in [tracing:true](https://www.apollographql.com/docs/apollo-server/api/apollo-server/#apolloserver).

![https://miro.medium.com/max/1400/1*u0oVzzJPCylM7aR47ZzzHA.png](https://miro.medium.com/max/1400/1*u0oVzzJPCylM7aR47ZzzHA.png)

[Profiling graphql gateway with Chrome DevTools](https://medium.com/@paul_irish/debugging-node-js-nightlies-with-chrome-devtools-7c4a1b95ae27) flame chart is not apparent if the function is small but there are many calls to it

Both turning on and removing them resulted in a 2x latency reduction from 700ms to 300ms for our test company. The reason at the time (as I understand) was that the time functions (like _performance.now()_) that measure spans for every resolver are too CPU-intensive.

[Ben here](https://www.youtube.com/watch?v=VnG7ej56lWw) did a nice benchmark of different backend servers and confirmed this.

## **Consider prefetching on frontend**

The timing of graphql query on a frontend is tricky. I wanted to move the initial graphql request as early as possible (before vendors.js) in the network waterfall. Doing this granted us some time, but it made the webapp much less maintainable.

To make the query, you need [graphql client](https://github.com/apollographql/apollo-client) and [gql literal parsing](https://github.com/apollographql/graphql-tag) and these typically would come via vendors.js. Now you would either need to bundle them separately or make a query with [raw fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch). Even if you do make a raw request, you’ll need to manage the response gracefully so that the response gets propagated into the correct models, (but those are initialized until later). So it made sense to not continue with this and maybe resort to server-side-rendering or [service workers](https://medium.com/samsung-internet-dev/a-beginners-guide-to-service-workers-f76abf1960f6) in the future.

![https://miro.medium.com/max/1400/1*GZSPDEh_sZrYsl2OHxxG_A.png](https://miro.medium.com/max/1400/1*GZSPDEh_sZrYsl2OHxxG_A.png)

Moving /graphql call earlier in the chain of network requests (Google Chrome inspector)

## **Evaluate query complexity**

What makes graphql unique from REST is that you can estimate how complicated a client’s request is going to be for your infrastructure before processing it. This is based purely on what he requests and how you define execution costs for your schema. If the estimated cost is too big, we reject the request, similar to our [rate limiting](https://pipedrive.readme.io/docs/core-api-concepts-rate-limiting).

We first tried [graphql-cost-analysis](https://github.com/pa-bru/graphql-cost-analysis) library but ended up creating our own because we wanted logic to take into account pagination multipliers, nesting, and impact types (network, I/O, DB, CPU). Though the hardest part is injecting custom cost directive into gateway & schema-registry. I hope we can opensource it too in the near future.

## **Schema has many faces**

Working with schema in js/typescript on a low level is confusing. You figure it out when you try to integrate federation into your existing graphql service.

For example, plain [koa-graphql](https://github.com/graphql-community/koa-graphql#options) and [apollo-server-koa](https://www.apollographql.com/docs/apollo-server/v1/servers/koa/) setups expect a nested [**GraphQLSchema**](https://github.com/graphql/graphql-js#using-graphqljs) param that includes resolvers, but federated apollo/server [wants schema to be passed separately](https://www.apollographql.com/docs/apollo-server/federation/implementing-services/#generating-a-federated-schema):

> buildFederatedSchema([{typeDefs, resolvers}])

In another case, you may want to define schema as an inline [gql tag](https://github.com/apollographql/graphql-tag) string or store it as schema.graphql file, but when you want to do cost evaluation, you may need it as [ASTNode](https://github.com/graphql/graphql-js/blob/master/src/language/ast.js) ([parse](https://github.com/graphql/graphql-js/blob/d4c82e0849318d045107321c6655c1a5da37b798/src/language/parser.d.ts#L58) / [buildASTSchema](https://github.com/graphql/graphql-js/blob/dd0297302800347a20a192624ba6373ee86836a3/src/utilities/buildASTSchema.js#L121)).

## **Gradual canary rollout**

During the mission, we did a gradual rollout to all internal **developers first** to catch obvious errors.

By the end of the mission, in February, we released graphql only to 100 lucky companies. We then slowly rolled it out to 1000 — 1%, 10%, 30%, 50%, and finally 100% of the customers finishing it in June.

The rollout was based on company ID and modulo logic. We also had allow- and deny-lists for test companies and for cases when developers didn’t want their companies to have graphql on yet. We also had an emergency off-switch that would revert it, which was handy during incidents to ease debugging

Considering how big of a change we did, it was a great way to get feedback and find bugs, while having lower risks for our customers.

# **Hopes & dreams**

To get all of the graphql benefits, we need to adopt mutations, **subscriptions,** and batch operations on a federated level. All of this needs teamwork & internal evangelism to expand the **number of federated services.**

Once graphql is stable and sufficient enough for our customers, it can become version 2 of our public API. A public launch would need directives to limit entity access based on [OAuth scopes](https://oauth.net/2/scope/) (for marketplace apps) and our [product suites](https://www.pipedrive.com/en/pricing).

For schema-registry, we need **tracking clients,** and getting **usage analytics** for better deprecation experience, **filtering**/highlighting of schemas’ costs & visibility, **naming validation**, managing non-ephemeral **persisted queries**, public schema **change history**.

As we have services in [go](https://golang.org/), it's unclear how internal communication should happen — over GRPC for speed & reliability, or individual graphql endpoints, or via centralized internal graphql gateway? If GRPC is better only due to its binary nature, could we make graphql binary instead with [msgpack](https://msgpack.org/)?

As for the outside world, I hope Apollo’s roadmap with [project Constellation](https://www.youtube.com/watch?v=MvHzOwdLb_o) will optimize _Query planner_ in [Rust](https://www.rust-lang.org/) so that we don’t see that 10% _gateway tax_ on performance, as well as enable flexible federation of services without their knowledge.

Exciting times to enjoy software development, full of complexity!