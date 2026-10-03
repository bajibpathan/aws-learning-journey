# 🍪 Elastic Load Balancing Sticky Sessions

> Sticky sessions allow a load balancer to bind a user's session to a specific backend instance instead of distributing that user's requests across different instances.

---

# 📖 Overview

Normally, a load balancer distributes incoming requests across multiple backend instances.

For example:

```text
                    Load Balancer
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
        Web Server 01           Web Server 02
```

As requests arrive, the load balancer distributes them across the available targets.

However, some applications require a user to continue communicating with the **same backend instance for the duration of a session**.

This is where **sticky sessions**, also known as **session affinity**, can be used.

---

# 🎯 What Is a Sticky Session?

Sticky sessions allow the load balancer to associate a client session with a particular target.

Without stickiness:

```text
User
 │
 ▼
Load Balancer
 │
 ├── Request 1 → Server 01
 ├── Request 2 → Server 02
 ├── Request 3 → Server 01
 └── Request 4 → Server 02
```

With stickiness enabled:

```text
User
 │
 ▼
Load Balancer
 │
 └──► Server 01
      ├── Request 1
      ├── Request 2
      ├── Request 3
      └── Request 4
```

The user's requests continue going to the same instance while the sticky session remains valid.

---

# 🛒 Why Use Sticky Sessions?

Consider an e-commerce application.

A customer:

1. Logs in.
2. Adds products to the shopping cart.
3. Continues browsing.
4. Proceeds toward checkout.

If session information is stored locally on a particular application server, moving the user between servers could affect that session information.

For example:

```text
User
 │
 ▼
Server 01
 │
 ├── Login Session
 └── Shopping Cart
```

The application may therefore require the user to continue communicating with:

```text
Server 01
```

for the lifetime of that session.

Sticky sessions provide a way to maintain this **session affinity**.

The transcript also notes that there are other ways to store session information, but stickiness is useful when the requirement is to keep a client associated with a particular instance.

---

# 🍪 How Stickiness Works

Stickiness uses a **cookie** to maintain the association between the client and the backend target.

Conceptually:

```text
Client
   │
   ▼
Load Balancer
   │
   ├── Set / Use Cookie
   │
   ▼
Target Instance
```

The cookie has an expiration period.

Depending on the application requirement, the session could remain sticky for a configured duration.

Once the sticky association is established:

```text
Client + Cookie
      │
      ▼
Load Balancer
      │
      ▼
Same Target
```

---

# ⚙️ Stickiness Options

The lesson discusses two approaches:

1. **Duration-based stickiness**
2. **Application-based stickiness**

---

# 1. Duration-Based Stickiness

With duration-based stickiness, the **load balancer manages the cookie**.

For example:

```text
Stickiness Duration = 5 minutes
```

The load balancer keeps the client associated with the target for that configured duration.

```text
Client
   │
   ▼
Load Balancer
   │
   │ Load Balancer Cookie
   ▼
Server 01
```

### Characteristics

- Managed by the load balancer
- Fixed stickiness duration
- Simple to configure
- Does not require application changes

### Useful When

The lesson describes this as useful when:

- The application is stateless.
- The session duration is predictable.
- Simple load-balancer-managed stickiness is sufficient.

### Limitation

The association persists for the configured duration even when the application may no longer require it.

---

# 2. Application-Based Stickiness

With application-based stickiness, the **application controls the cookie**.

```text
Application
    │
    ▼
Custom Cookie
    │
    ▼
Load Balancer
    │
    ▼
Maintain Session Affinity
```

The application can determine how long the session should remain sticky.

This provides more control over situations such as:

- User logout
- User inactivity
- Application-specific session lifetime

### Advantages

- Greater flexibility
- Fine-grained session control
- Application can expire the session when required

### Considerations

- Requires application changes
- More complex to implement than load-balancer-generated stickiness

---

# 📊 Stickiness Options Comparison

| Feature | Duration-Based | Application-Based |
|---|---|---|
| Cookie controlled by | Load balancer | Application |
| Duration | Fixed/configured | Application controlled |
| Application changes | Not required | Required |
| Complexity | Lower | Higher |
| Flexibility | Lower | Higher |
| Best fit | Predictable session duration | Application-specific session control |

A simple way to remember the difference:

```text
Load Balancer decides duration
            │
            ▼
     Duration-Based


Application controls session
            │
            ▼
    Application-Based
```

---

# 🧪 Console Demonstration

The lesson demonstrates stickiness using an existing Application Load Balancer with two backend web servers.

Each server displays a different page so that traffic distribution can be observed visually.

Without stickiness:

```text
Refresh
   │
   ├──► Orange Server
   │
   ├──► Blue Server
   │
   ├──► Orange Server
   │
   └──► Blue Server
```

The browser can therefore reach different backend instances as requests are distributed.

---

# Step 1: Open the Target Group

Navigate to:

**EC2 → Target Groups**

Select the target group associated with the Application Load Balancer.

Choose:

**Actions → Edit target group attributes**

---

# Step 2: Enable Stickiness

Locate the traffic configuration and enable:

```text
Stickiness
```

For the demonstration, select:

```text
Load Balancer Generated Cookie
```

and configure:

```text
Duration = 1 minute
```

Save the changes.

---

# Step 3: Test Stickiness

Return to the application.

Suppose the browser currently reaches:

```text
Web Server 01
Orange Page
Availability Zone 1A
```

Refresh the browser several times.

Instead of switching between the two servers, requests should continue reaching the same server while the sticky session remains valid.

```text
Refresh 1 → Server 01
Refresh 2 → Server 01
Refresh 3 → Server 01
Refresh 4 → Server 01
```

This demonstrates that session affinity is working.

---

# 🔍 Inspect the Cookie

The lesson also demonstrates the cookie using browser Developer Tools.

Open:

**Browser Developer Tools → Network**

Refresh the application and inspect the request/response headers.

The browser should show the cookie created for the load-balancer session.

The cookie also contains information associated with its expiration.

Conceptually:

```text
Browser
   │
   │ Cookie
   ▼
Application Load Balancer
   │
   ▼
Previously Selected Target
```

The load balancer uses this mechanism to keep subsequent requests associated with the same backend target.

---

# Step 4: Disable Stickiness

Return to:

**EC2 → Target Groups → Edit target group attributes**

Turn off:

```text
Stickiness
```

Save the changes.

Allow a short period for the change to take effect.

Refresh the application again.

You should once again observe requests being distributed between the backend servers.

```text
Refresh 1 → Server 01

Refresh 2 → Server 02

Refresh 3 → Server 01

Refresh 4 → Server 02
```

---

# ⚠️ Important Considerations

## Stickiness Changes Normal Traffic Distribution

Without stickiness, requests can be distributed among the available targets.

With stickiness:

```text
User A → Server 01
User A → Server 01
User A → Server 01
```

The purpose is to preserve the relationship between that user session and the selected target.

---

## Cookie Expiration Matters

Sticky sessions are not necessarily permanent.

The cookie has an expiration duration.

```text
Sticky Session
      │
      ▼
Cookie Valid
      │
      ▼
Same Target
      │
      ▼
Cookie Expires
```

The duration should therefore be chosen according to the application's session requirements.

---

## Sticky Sessions Are Not the Only Way to Manage Session Data

The lesson specifically notes that applications can use other approaches for storing session information.

Sticky sessions are useful when the requirement is specifically:

> Keep this client associated with the same backend instance for the session.

---

# ❓ Interview Questions

### Q1. What is a sticky session?

A sticky session allows a load balancer to bind a client's session to a particular backend instance for the duration of the sticky session.

### Q2. What is another name for sticky sessions?

**Session affinity.**

### Q3. Why might an application require sticky sessions?

An application may need a user to remain connected to the same instance so that session-related information is maintained consistently during that session.

### Q4. Give an example use case.

An e-commerce application where a user logs in, adds products to a shopping cart, and needs to maintain the same session while continuing to use the application.

### Q5. How does the load balancer maintain stickiness?

Using cookies that associate the client's session with a particular target.

### Q6. What are the two stickiness approaches discussed in the lesson?

- Duration-based stickiness
- Application-based stickiness

### Q7. Who controls duration-based stickiness?

The load balancer.

### Q8. Who controls application-based stickiness?

The application.

### Q9. What is an advantage of duration-based stickiness?

It is simple to configure and does not require application changes.

### Q10. What is an advantage of application-based stickiness?

It provides greater flexibility and allows the application to control the session lifetime.

### Q11. What is a disadvantage of application-based stickiness?

It requires application changes and is more complex to implement.

### Q12. What happens after stickiness is disabled?

The load balancer can again distribute the client's requests among the available backend targets instead of maintaining the sticky association.

---

# 💡 Key Takeaways

- Load balancers normally distribute requests across multiple backend targets.
- Some applications require a user to remain associated with one backend instance.
- This behavior is called **sticky sessions** or **session affinity**.
- Stickiness uses cookies to maintain the client-to-target association.
- Cookies have an expiration duration.
- Duration-based stickiness is managed by the load balancer.
- Application-based stickiness is controlled by the application.
- Load-balancer-managed stickiness is simpler to configure.
- Application-based stickiness provides greater session control but requires application changes.
- E-commerce sessions are one example where maintaining session affinity may be useful.
- Stickiness can be enabled and disabled through the target group's attributes.

The core concept to remember is:

```text
WITHOUT STICKINESS

User
 │
 ▼
Load Balancer
 │
 ├──► Server 01
 ├──► Server 02
 └──► Server 01


WITH STICKINESS

User
 │
 │ Cookie
 ▼
Load Balancer
 │
 └──► Server 01
       ↑
       │
   Same target
   during session
```

---

# 📚 Related Topics

- Elastic Load Balancing
- Application Load Balancer
- Target Groups
- Load Balancer Cookies
- Application Cookies
- Session Affinity
- EC2 Auto Scaling
- Load Balancer Health Checks