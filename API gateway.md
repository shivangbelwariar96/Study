This video has a clear arc: define what an API gateway actually does (and what separates it from a load balancer), walk through its major capabilities one by one, then zoom out to show how a "single entry point" can still handle millions of requests per second through multi-region/multi-AZ deployment. Let me cover every piece with mobile-friendly visuals.

## Part 1 — What an API gateway is, in one sentence

An API gateway **accepts client API requests and routes them to the correct backend service based on the API endpoint**. That sentence is doing all the heavy lifting — read it twice, because every word matters. It accepts requests (it sits at the edge, in front of everything). It looks at the API endpoint (it understands HTTP-level details). It routes (it decides which service handles which request).

<img width="1472" height="760" alt="image" src="https://github.com/user-attachments/assets/88fa54af-ffcf-4337-99a8-86ad556b9923" />


## Part 2 — Why this isn't a load balancer (the most-confused point)

A load balancer's job is **to distribute traffic evenly across multiple instances of the same service**. If you have three identical instances of the invoice microservice, the load balancer is what picks one of them per request — round-robin, least-connections, whatever algorithm. It does not look at *what* the request is. It cannot tell `/invoice` from `/order`. It just spreads load.

An API gateway, by contrast, **reads the request**. It understands `/invoice` means invoice service, `/order` means order service. It chooses the destination *service*, not the destination *instance*. The two layers do completely different jobs and they typically work together.The order is: **client → API gateway → load balancer → instance**. The gateway picks the destination service; the load balancer for that service picks one of its instances.

<img width="1472" height="760" alt="image" src="https://github.com/user-attachments/assets/00e938ad-aa93-4952-9ba1-4a70f7a93755" />


## Part 3 — Capability #1: API composition

The first major capability beyond routing. Imagine the "View my orders" screen in an e-commerce app. On **mobile**, you have limited bandwidth and limited screen space, so you show just two things — product details and invoice details. On a **PC**, you have room for more, so you also show ratings, reviews, and recommendations.

Without an API gateway, the client has to make that decision itself — figure out which device it's on, then call two different microservices on mobile and four or five on PC, then assemble the results. That's complicated client code, with version skew problems (when you change which APIs are needed, you have to ship a new mobile app).

With an API gateway, the client just calls one endpoint — `/api/myorder` — and the gateway figures out the rest. It detects the device (or accepts a hint from the client), calls the right backend services in parallel, gathers the results into one JSON response, and returns it. The client gets a single payload.The speaker calls out Netflix specifically — their entire architecture leans on this pattern, with different gateway endpoints tuned to different device classes (TV, mobile, web, gaming console). One client call, one device-tailored response.

<img width="1472" height="920" alt="image" src="https://github.com/user-attachments/assets/536033cf-eb66-4146-80b4-2847b5f888a9" />


## Part 4 — Capability #2: Authentication

The second capability is one most production systems care about deeply. You don't want to put auth logic inside every microservice — that's duplication, and it's a footgun (what if one service forgets a check?). Centralize it at the gateway.

The flow (using OAuth 2.0 as the example):

1. Client first hits the **authorization server** with credentials and gets back an **access token** (typically a JWT).
2. Client then makes its real API request to the **API gateway**, passing the access token in the `Authorization: Bearer <token>` header.
3. The gateway checks the token's validity — either by verifying its signature locally (the fast path, if it's a JWT) or by calling the authorization server's introspection endpoint.
4. If valid, the gateway forwards the request to the appropriate microservice. If not, it returns `401 Unauthorized` and the request never reaches the backend.

<img width="1472" height="920" alt="image" src="https://github.com/user-attachments/assets/c3c6c769-1dc1-4bbe-b1e4-1ad2a4e0efcd" />

## Part 5 — Capability #3: Rate limiting

The third capability has three sub-pieces that the speaker treats separately: burst limits, throttling, and queues.

**Burst limit** — the most coarse-grained. Sets the maximum number of concurrent in-flight requests the gateway will accept at peak. If you cross it, the gateway returns `429 Too Many Requests`. On AWS/Azure API gateway you literally set a numeric value (say, 500). This protects everything downstream from being overwhelmed during a spike.

**Throttling** — more granular. Per-API or per-user rules. Examples:
- `/api/invoice` may be called at most 10 times per minute (globally).
- A specific user may call any API at most 100 times per minute.
- A specific IP address may be blocked entirely.

The same mechanism handles abuse prevention (blocking attackers) and fair-use enforcement (preventing one customer from monopolizing capacity).

**API queue** — once a request exceeds the immediate burst limit, instead of rejecting it outright, the gateway holds it in a waiting area. When capacity frees up, queued requests are processed in order. This smooths out bursts that briefly exceed steady-state capacity. The speaker explicitly ties this back to the **thundering herd** problem — the queue is your safety valve for synchronized traffic spikes.

<img width="1472" height="920" alt="image" src="https://github.com/user-attachments/assets/d8cf684b-50c0-4d93-8404-0fda40a4f919" />


## Part 6 — Capability #4: Service discovery

In a distributed system, microservice instances are constantly being created and destroyed by autoscaling. The IP addresses and ports of those instances change continuously. The gateway can't hard-code where to send `/invoice` requests — by the time the config was loaded, those addresses are wrong.

**Service discovery** is the registry that solves this. Two approaches:

1. **Self-registration.** When a microservice instance starts up, it calls the service discovery server and registers itself: "I'm `invoice-svc`, my address is `10.0.5.23:8080`, I'm available." When it shuts down, it deregisters. This requires the service code to know about the discovery system.

2. **Health-check based.** The service discovery system periodically pings every registered instance. If an instance stops responding (no heartbeat), discovery automatically removes it from the active list. Simpler for the service code; more network chatter from discovery.

Either way, when the gateway needs to forward a request, it asks discovery: *"Give me the location of `invoice-svc`."* Discovery returns one of the currently active instances (or the address of the load balancer fronting them). The gateway forwards the request there.

<img width="1472" height="920" alt="image" src="https://github.com/user-attachments/assets/97d6b356-06ec-422a-9b07-42ed9ef4b766" />


**Tool name correction:** the speaker mentions "Zuul and Udega" — that's a transcription artifact. The actual tools are **Zuul** (Netflix's API gateway, which has some discovery integration) and **Eureka** (Netflix's dedicated service discovery server). Consul (HashiCorp) is the other common name in this space.

## Part 7 — Other capabilities (briefly)

The four above are the major ones, but the speaker notes additional capabilities that come standard with most production API gateways:

- **Request/response transformation** — modify the shape of requests or responses in flight (rename fields, version-translate APIs, strip internal-only fields before they leave the system).
- **Response caching** — for idempotent endpoints, cache the response so the next identical request can be answered without invoking the backend at all.
- **Logging and observability** — centralized place to log every request, useful for analytics, debugging, and audit trails.

## Part 8 — The pivot: how is this still "a single entry point" at scale?

The speaker now poses the central question this video is built around: **if the API gateway is a single entry point, how does it handle millions of requests per second without being a single point of failure?**

The answer takes the rest of the video, and it requires understanding cloud regions and availability zones first.

### Regions and availability zones

A **region** is a geographic area — Mumbai, Chennai, Frankfurt, Virginia. Cloud providers (AWS, Azure, GCP) operate data centers in many regions.

Inside each region there are multiple **availability zones (AZs)** — physically isolated facilities within the same metro area. Within Mumbai, AZ1 might be in Bandra and AZ2 might be in some other neighborhood. Each AZ has its own buildings, power, cooling, and network. They share nothing physical.

The point: if AZ1's power goes out, AZ2 keeps running. If a flood hits Bandra, the other AZ is unaffected. You need both AZs in a region to fail for the region to be considered down. And you need multiple regions to fail for the whole service to go down.

<img width="1472" height="760" alt="image" src="https://github.com/user-attachments/assets/8a432c64-dbb8-47ba-8977-b27748fbf1c5" />


## Part 9 — The full multi-AZ, multi-region picture

Now the pieces snap together. Inside each AZ, you have the full stack: load balancers, microservice instances, multiple instances of each service. One API gateway sits at the region level and routes traffic into the appropriate AZ.

The complete hierarchy from outside to inside:

1. **DNS** — picks a region.
2. **API gateway (per region)** — picks an AZ, then picks a service.
3. **Load balancer (per service, per AZ)** — picks an instance.
4. **Microservice instance** — does the work.

<img width="1472" height="1160" alt="image" src="https://github.com/user-attachments/assets/c72f1d62-a386-4c4e-b555-5f42bf28b9d0" />

**No single point of failure** at any layer:

- An instance dies → the LB sends traffic to its siblings.
- An entire AZ dies → the regional gateway sends traffic to the other AZ.
- An entire region dies → DNS sends traffic to another region.
- DNS dies → see Part 11.

## Part 10 — DNS-based load balancing across regions

The piece that ties multiple regions together is the **DNS-based load balancer** — what AWS calls **Route 53** and Azure calls **Traffic Manager**. When a client looks up `example.com`, the DNS resolver doesn't just return one fixed IP. It returns an IP **chosen intelligently** based on several criteria:

- **Latency** — pick the region closest to the client (lowest round-trip time).
- **Health** — skip regions that are currently failing health checks.
- **Compliance** — some jurisdictions require data to stay within the country. EU customer traffic might be required to go to an EU region even if a US region is closer.
- **Weight** — for canary deployments or A/B testing, send some percentage to each region.

So the same `example.com` resolves differently depending on who's asking. A user in India gets the Mumbai API gateway's IP; a user in Singapore gets Singapore's; a user in Frankfurt gets Frankfurt's.

This is the layer that makes the "single entry point" *feel* like one entry point to clients (they all type the same URL) while actually being many independent gateways behind the scenes.

<img width="1472" height="920" alt="image" src="https://github.com/user-attachments/assets/d882af17-68b8-4913-9167-3f0f1114c563" />


## Part 11 — Is DNS itself a single point of failure?

The speaker raises this final natural question: if everything depends on DNS to find the regional gateway, and DNS goes down, doesn't the whole system collapse?

**No** — and this is worth understanding briefly. DNS is itself **distributed and hierarchical**. When you type `example.com`, the resolution doesn't happen at a single server. It walks a tree:

- Your **local DNS resolver** (often run by your ISP, or 8.8.8.8 from Google, or 1.1.1.1 from Cloudflare).
- The **root DNS** servers (13 logical roots, hundreds of physical instances worldwide via anycast).
- The **TLD** DNS servers (responsible for `.com`, `.org`, etc.).
- The **authoritative DNS** servers (run by the domain owner — for example, AWS Route 53 if the domain uses Route 53).

Each layer has many redundant servers, and the answers get cached at every level. The system is engineered to survive any individual node failing. DNS as a whole is one of the most resilient systems on the internet — far more resilient than any single application infrastructure.

The speaker defers a deep DNS dive to a future video; the short version is that "DNS goes down" is essentially never a one-server event, it's an internet-scale event, and the systems we're discussing all have bigger problems if that happens.

## Part 12 — The mental model in one paragraph

An API gateway is the intelligent edge layer of a microservices architecture — it accepts every external request, **understands HTTP and the API endpoint**, and routes the request to the correct backend service (distinct from a load balancer, which only spreads load across instances of *one* service). On top of routing it adds **API composition** (one client call fans out to multiple backend services, with the gateway assembling the response — heavily used by Netflix for device-specific payloads), **centralized authentication** (verify the OAuth token once at the gateway so individual services don't each implement auth), **rate limiting** (burst limits to cap peak concurrency, throttling for per-API/per-user/per-IP rules, queues to smooth out thundering-herd bursts), and **service discovery** (look up live instance addresses via Zuul/Eureka/Consul so the gateway doesn't hard-code locations that autoscaling makes stale). To handle millions of requests per second without becoming a single point of failure, the gateway itself is replicated: multiple instances per **availability zone**, multiple AZs per **region** (each AZ a physically isolated data center with its own power and network), and multiple regions globally — with a **DNS-based load balancer** (Route 53, Traffic Manager) at the very top distributing client traffic across regions by latency, health, and compliance rules. The result is one URL that feels like a single entry point but is actually a tree of redundant gateways, where every layer can lose a node and the system keeps serving traffic.

If you want, two natural follow-ups the speaker only briefly mentions:

**DNS deep-dive** — the hierarchy from local resolver to root to TLD to authoritative servers, why it's so hard to take down, and how things like anycast routing make 13 "root servers" actually answer queries from hundreds of physical locations.

**Service mesh vs API gateway** — once you have service-to-service traffic inside the cluster (not just client-to-edge), the question becomes how those internal calls get their service discovery, retries, and observability. That's what Istio/Linkerd/Consul Connect solve, and the answer is "sidecar proxies", a very different shape from a centralized gateway.
