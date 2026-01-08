# Why You Should Upgrade Your Blazor App to .NET 10

*Scope: Blazor Server, Blazor Web Apps, and Blazor WebAssembly on ASP.NET Core*

This is an adjunct document to an interview I did with Daniel Roth from the Microsoft ASP.NET Core Team on BlazorTrain episode 109:

YouTube Video: https://youtu.be/ydd13J1Vz2M

BlazorTrain Home Page: https://blazortrain.com

---

## TL;DR for developers

If you’re running Blazor on **.NET 6, 7, 8, or 9**, upgrading to **.NET 10** gives you:

- **Measurable performance gains** in ASP.NET Core and the .NET runtime (often without code changes).
- **Lower memory usage and fewer allocations**, which directly improves scalability.
- **Better reliability for Blazor Server apps**, especially around reconnection and lost state.
- **Stronger diagnostics and observability**, making production issues easier to find and fix.
- A **longer support runway** and a platform that matches where Blazor development is clearly headed.

This document walks through *what actually improves at each version step*, using **only published Microsoft numbers where they exist**, and explains why those improvements matter to you as a Blazor developer.

---

## The core idea: Blazor performance is stack performance

Blazor apps don’t run in isolation. Even if your UI logic is already efficient, your app still depends on:

- Kestrel
- Routing and middleware
- Serialization
- Thread pool behavior
- GC and allocation patterns
- Diagnostics and logging

Microsoft explicitly treats ASP.NET Core and Blazor as part of the same performance story, and improvements to the hosting stack translate directly into better Blazor behavior under load ([ASP.NET Core 7 performance overview](https://devblogs.microsoft.com/dotnet/performance-improvements-in-aspnet-core-7/#:~:text=Performance%20is%20a%20feature%20of%20.NET)).

Upgrading your runtime is often the **highest ROI performance change you can make**, because you get the gains *without touching your app code*.

---

## What you gain by upgrading (the strongest arguments)

### 1. Huge server-side throughput improvements (.NET 6 → 7)

Microsoft published dramatic gains in ASP.NET Core 7 on large-core machines:

- **Plaintext benchmark:** +514% throughput (2.4M → 14.6M RPS)  
- **JSON benchmark:** +311% throughput (270K → 1.1M RPS)  

These gains came from reducing contention in the thread pool and socket memory pool ([source](https://devblogs.microsoft.com/dotnet/performance-improvements-in-aspnet-core-7/#:~:text=Partitioning%20enables%20cores%20to%20operate%20on%20their%20own%20queues%2C%20which%20helps%20reduce%20contention%20and%20improve%20scalability)).

**Why this matters to you:**  
Blazor Server apps are concurrency-heavy. Each connected user is a live circuit. Improvements in scheduling and contention directly affect how many concurrent users you can support before latency spikes.

---

### 2. Real-world benchmark wins (.NET 7 → 8)

Microsoft reports that **ASP.NET Core in .NET 8** is:

- **18% faster** on the TechEmpower JSON benchmark
- **24% faster** on the TechEmpower Fortunes benchmark

compared to .NET 7 ([source](https://devblogs.microsoft.com/dotnet/announcing-asp-net-core-in-dotnet-8/#:~:text=Compared%20to%20.NET%207%2C%20ASP.NET%20Core%20in%20.NET%208%20is%2018%25%20faster%20on%20the%20Techempower%20JSON%20benchmark%20and%2024%25%20faster%20on%20the%20Fortunes%20benchmark)).

**Why this matters to you:**  
These are full-stack web benchmarks, not micro-tests. Even if your Blazor UI isn’t CPU-bound, faster request handling improves page loads, API calls, authentication flows, and background operations.

---

### 3. Lower allocation costs and faster execution (.NET 9 → 10)

In a published .NET 10 runtime benchmark involving delegates and closures, Microsoft shows:

- Execution time reduced by **~66%** (19.53 ns → 6.69 ns)
- Allocations reduced by **~73%** (88 B → 24 B)

([source](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-10/#:~:text=Sum%20.NET%209.0%2019.530%20ns)).

Microsoft frames this as reducing the long-standing “abstraction penalty” in .NET ([source](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-10/#:~:text=.NET%20historically%20has%20had%20an%20%E2%80%9Cabstraction%20penalty)).

**Why this matters to you:**  
Blazor code is inherently abstracted (components, render trees, delegates, lambdas). Lower allocation pressure means:

- Fewer GC pauses
- Better tail latency
- Higher user density per server

---

### 4. Blazor Server reliability improvements (.NET 9 → 10)

.NET 10 introduces **circuit state persistence** for Blazor Web Apps. If a user’s connection drops (mobile backgrounding, network loss), their session can be resumed **without losing unsaved state**.

([source](https://learn.microsoft.com/en-us/aspnet/core/release-notes/aspnetcore-10.0?view=aspnetcore-10.0#:~:text=Blazor%20Web%20Apps%20can%20now%20persist%20a%20user's%20session%20(circuit)%20state)).

**Why this matters to you:**  
This directly reduces:

- Rage-refreshes
- Lost form data
- Support tickets blaming “Blazor disconnected again”

It’s a UX and reliability feature that directly impacts perception of your app.

---

### 5. Better diagnostics and observability (.NET 9 → 10)

.NET 10 adds:

- Built-in metrics that include **Blazor**
- Improved **Blazor Server tracing**
- New diagnostic tools for **Blazor WebAssembly** (CPU profiles, memory dumps)

([metrics](https://devblogs.microsoft.com/dotnet/announcing-dotnet-10/#:~:text=New%20Built-in%20Metrics),  
[tracing](https://devblogs.microsoft.com/dotnet/announcing-dotnet-10/#:~:text=Improved%20Blazor%20Tracing),  
[WASM tools](https://devblogs.microsoft.com/dotnet/announcing-dotnet-10/#:~:text=Blazor%20WebAssembly%20Diagnostic%20Tools)).

**Why this matters to you:**  
Performance bugs you can’t observe are performance bugs you can’t fix. These tools reduce time-to-root-cause in production.

---

## Version-by-version summary

### .NET 6 → 7

- Massive throughput gains on modern hardware
- Much better scalability under concurrency
- Strong runtime improvements via dynamic PGO

### .NET 7 → 8

- Double-digit percentage gains in real web benchmarks
- Introduction of native AOT for ASP.NET Core (important for API-heavy Blazor solutions)

### .NET 8 → 9

- Higher throughput, faster startup, and lower memory usage (Microsoft product claim)
- Blazor-specific performance improvements and better reconnection UX

### .NET 9 → 10

- Significant runtime-level allocation and execution improvements
- Circuit state persistence for Blazor Server
- Stronger diagnostics across Server and WASM

---

## About “6 → 10 is X% faster” claims

There is **no single official number** for “.NET 6 → .NET 10 speedup,” because Microsoft publishes results across different benchmark types and scenarios.

The correct developer approach is:

1. Use Microsoft’s published benchmarks to justify the upgrade direction.
2. Benchmark *your own app* on two runtimes to get defensible numbers.

Microsoft uses BenchmarkDotNet for their own analysis and recommends similar approaches ([source](https://devblogs.microsoft.com/dotnet/performance-improvements-in-aspnet-core-7/#:~:text=We%20will%20use%20BenchmarkDotNet)).

---


---

## What improves specifically for Blazor WebAssembly (WASM)

Blazor WebAssembly apps benefit primarily from **runtime**, **tooling**, and **diagnostics** improvements rather than raw server throughput. From .NET 6 through .NET 10, Microsoft has steadily reduced download size, startup cost, runtime overhead, and improved your ability to debug real-world WASM issues.

### 1. Smaller downloads and faster startup (runtime & SDK evolution)

Microsoft has consistently focused on reducing WebAssembly payload size and startup time across releases by:

- improving IL trimming,
- optimizing runtime packing,
- reducing framework and runtime overhead.

While Microsoft does not publish a single rolled-up “WASM startup % improvement” across versions, each release explicitly lists size and startup work as part of the Blazor and runtime roadmap (example framing in .NET 9 and .NET 10 announcements).  
([.NET 9 announcement – Blazor focus](https://devblogs.microsoft.com/dotnet/announcing-dotnet-9/#:~:text=performance%20improvements%20across%20all%20areas%20of%20Blazor)).

**Why this matters to you:**  
Smaller downloads and faster startup directly impact:

- cold-load time on mobile networks,
- perceived responsiveness for first-time users,
- CDN and bandwidth costs at scale.

---

### 2. Runtime execution improvements apply directly to WASM

The .NET runtime improvements described for .NET 9 and .NET 10 are **not server-only**. Allocation reductions and faster execution paths benefit:

- Blazor WebAssembly component rendering,
- event handling,
- JSON serialization/deserialization,
- client-side validation and state management.

For example, Microsoft shows **~66% execution time reduction** and **~73% allocation reduction** in a .NET 10 runtime benchmark vs .NET 9 for delegate-heavy code paths ([source](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-10/#:~:text=Sum%20.NET%209.0%2019.530%20ns)).  

**Why this matters to you:**  
Blazor WASM apps frequently rely on delegates, lambdas, and abstractions in UI and state logic. Lower allocation pressure:

- improves frame stability,
- reduces GC pauses in the browser,
- keeps UI interactions smoother on lower-end devices.

---

### 3. Improved diagnostics for Blazor WebAssembly (.NET 10)

.NET 10 significantly improves WASM diagnostics by adding:

- **CPU profiling**
- **memory dump support**
- **runtime metrics**

for Blazor WebAssembly apps ([source](https://devblogs.microsoft.com/dotnet/announcing-dotnet-10/#:~:text=Blazor%20WebAssembly%20Diagnostic%20Tools)).

**Why this matters to you:**  
Prior to these tools, diagnosing real-world WASM performance issues often meant guessing or reproducing locally. These additions allow you to:

- identify hot paths in client-side code,
- find memory leaks caused by retained component state,
- debug production-only issues with far more confidence.

---

### 4. Better parity between Server and WASM mental models

Recent Blazor releases emphasize **shared component models** and consistent behaviors across:

- Blazor Server
- Blazor Web Apps
- Blazor WebAssembly

Microsoft explicitly positions Blazor as a unified UI framework that benefits from improvements across hosting models ([Full-stack Blazor framing](https://devblogs.microsoft.com/dotnet/announcing-dotnet-9/#:~:text=Full%20stack%20web%20development%20with%20ASP.NET%20Core%20%26%20Blazor)).

**Why this matters to you:**  
If you maintain multiple hosting models (or may switch later), upgrading reduces divergence and technical debt between WASM and Server builds.

---

### 5. Native AOT groundwork (important for future WASM performance)

.NET 8 introduces native AOT for ASP.NET Core, and Microsoft continues laying groundwork for broader AOT scenarios ([source](https://devblogs.microsoft.com/dotnet/announcing-asp-net-core-in-dotnet-8/#:~:text=native%20AOT%20support)).

While Blazor WebAssembly AOT has existed earlier, ongoing runtime and toolchain improvements improve:

- build reliability,
- compatibility with trimming,
- real-world viability of AOT-enabled WASM apps.

**Why this matters to you:**  
As WASM-heavy apps grow more complex, AOT becomes increasingly important for startup time and runtime predictability.

---


---

## When upgrading matters *less* (and how to think about it honestly)

Being explicit about where upgrades deliver **smaller or indirect gains** builds credibility with developers. There *are* scenarios where moving to .NET 10 won’t immediately show dramatic wins—and that’s okay.

### 1. Very small, low-traffic apps

If your Blazor app:

- has a small user base,
- runs on lightly loaded hardware,
- and rarely experiences concurrent usage,

you may not *feel* the throughput and allocation improvements right away.

**Why this is still relevant:**  
Even low-traffic apps benefit from:

- reduced memory usage over time,
- improved diagnostics when issues do occur,
- staying on a supported, modern runtime.

Microsoft’s performance work is largely about **scalability and efficiency under load**, which may not surface immediately in small deployments ([ASP.NET Core scalability framing](https://devblogs.microsoft.com/dotnet/performance-improvements-in-aspnet-core-7/#:~:text=reduce%20contention%20and%20improve%20scalability)).

---

### 2. UI-bound or network-bound workloads

If your Blazor app is primarily limited by:

- slow backend APIs,
- third-party services,
- client-side network latency,

runtime-level performance improvements may not be the dominant factor in end-to-end latency.

**Why upgrading still helps:**  

- Faster serialization/deserialization
- Lower allocation pressure during UI updates
- Better tooling to identify *where* the real bottleneck lives

Microsoft’s diagnostics improvements in .NET 10 are explicitly designed to help identify these issues rather than guess ([.NET 10 observability focus](https://devblogs.microsoft.com/dotnet/announcing-dotnet-10/#:~:text=Enhanced%20Observability%20and%20Diagnostics)).

---

### 3. Pure static-content scenarios (especially WASM)

If your Blazor WebAssembly app:

- loads once,
- mostly displays static content,
- performs minimal client-side logic,

you may not see dramatic runtime execution improvements.

**Why this still doesn’t argue against upgrading:**  

- Startup size and runtime efficiency improvements compound over releases.
- Diagnostics improvements matter when issues *do* occur.
- Future features and fixes increasingly target newer runtimes.

Microsoft positions Blazor as an evolving platform, with improvements landing continuously rather than only in major architectural changes ([Blazor roadmap framing](https://devblogs.microsoft.com/dotnet/announcing-dotnet-9/#:~:text=performance%20improvements%20across%20all%20areas%20of%20Blazor)).

---

### 4. Apps near end-of-life

If an app is:

- scheduled for retirement,
- frozen with no new features,
- or being replaced entirely,

the business case for upgrading purely for performance may be weaker.

**However:**  
Even here, upgrading may still be justified to:

- stay within supported .NET versions,
- reduce security risk,
- simplify infrastructure standardization.

Microsoft maintains defined support lifecycles for .NET versions, and older runtimes eventually exit support ([.NET support policy](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)).

---

### 5. The honest trade-off: upgrade cost vs. payoff

Upgrading is not free. You may encounter:

- minor breaking changes,
- dependency updates,
- test updates.

But Microsoft has consistently focused on **compatibility and incremental migration**, especially across LTS releases ([.NET compatibility approach](https://learn.microsoft.com/en-us/dotnet/core/compatibility/)).

For most actively maintained Blazor apps, the cost is usually **front-loaded and finite**, while the performance, reliability, and tooling benefits accumulate over time.

---

## How to use this section in real conversations

When discussing upgrades with your team, this section helps you say:

- “We don’t expect miracles in *this* area…”
- “…but we *do* get better diagnostics, lower risk, and future-proofing.”
- “If we ever scale or add features, we’ll already be on the right runtime.”

That honesty tends to **lower resistance**, not increase it.

## WASM-specific bottom line for developers

Upgrading Blazor WebAssembly apps to **.NET 10** gives you:

- Faster client-side execution from runtime improvements
- Lower allocation pressure in UI-heavy code
- Better startup characteristics over time
- Real diagnostics tools instead of guesswork
- A future-proof platform aligned with Microsoft’s WASM investment

Even when server-side gains don’t apply, **WASM apps still benefit materially from upgrading**—especially in responsiveness, debuggability, and long-term maintainability.

## Bottom line for developers

Upgrading Blazor to **.NET 10** isn’t about chasing version numbers. It’s about:

- Getting **free performance** from the runtime and framework
- Reducing memory pressure and GC overhead
- Making Blazor Server apps more resilient
- Having modern diagnostics when things go wrong

If you’re already maintaining the app, the upgrade cost is usually small compared to the cumulative performance, reliability, and operability gains.

---

## Primary sources

- https://devblogs.microsoft.com/dotnet/performance-improvements-in-aspnet-core-7/
- https://devblogs.microsoft.com/dotnet/announcing-asp-net-core-in-dotnet-8/
- https://devblogs.microsoft.com/dotnet/announcing-dotnet-9/
- https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-10/
- https://devblogs.microsoft.com/dotnet/announcing-dotnet-10/
- https://learn.microsoft.com/en-us/aspnet/core/release-notes/aspnetcore-10.0?view=aspnetcore-10.0
