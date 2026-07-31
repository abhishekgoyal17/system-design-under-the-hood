<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Backend Under Load: Eight Failure Modes</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=Source+Serif+4:opsz,wght@8..60,400;8..60,500;8..60,600;8..60,700&family=IBM+Plex+Mono:wght@400;500;600&display=swap" rel="stylesheet">
<style>
  :root{
    --paper:#F4F5F0;
    --paper-2:#ECEEE7;
    --ink:#181C22;
    --ink-soft:#565F6B;
    --line:#D8DBD1;
    --panel:#171B22;
    --panel-2:#20252E;
    --panel-text:#E9EBEE;
    --red:#C7454A;
    --red-bg:#F7E4E3;
    --amber:#C8862B;
    --amber-bg:#F6E9D3;
    --green:#2F8F63;
    --green-bg:#DDEFE4;
    --blue:#2C5FD1;
    --blue-bg:#DFE6F9;
    --radius:10px;
    --maxw:760px;
  }
  *{box-sizing:border-box;}
  html{scroll-behavior:smooth;}
  body{
    margin:0;
    background:var(--paper);
    color:var(--ink);
    font-family:'Source Serif 4', Georgia, serif;
    font-size:18px;
    line-height:1.65;
    -webkit-font-smoothing:antialiased;
  }
  h1,h2,h3,h4,.eyebrow,.mono-label,nav,.btn,.diagram-title,.chip{
    font-family:'Space Grotesk', sans-serif;
  }
  code, pre, .mono, .cheat-table td, .cheat-table th{
    font-family:'IBM Plex Mono', monospace;
  }
  a{color:var(--blue);}
  a:hover{color:var(--ink);}

  .signal{display:inline-flex; gap:5px; align-items:center;}
  .signal span{width:9px;height:9px;border-radius:50%;display:inline-block; box-shadow:0 0 0 1px rgba(0,0,0,.08) inset;}
  .signal .r{background:var(--red);} .signal .a{background:var(--amber);} .signal .g{background:var(--green);}

  .topnav{
    position:sticky; top:0; z-index:50;
    background:rgba(244,245,240,.88);
    backdrop-filter:blur(8px);
    border-bottom:1px solid var(--line);
    padding:14px 24px;
    display:flex; align-items:center; justify-content:space-between;
  }
  .topnav .brand{
    font-weight:600; font-size:14px; letter-spacing:.02em;
    display:flex; gap:8px; align-items:center;
    color:var(--ink);
    text-decoration:none;
  }
  .topnav a.back{font-size:14px; text-decoration:none; color:var(--ink-soft); font-family:'Space Grotesk',sans-serif;}
  .topnav a.back:hover{color:var(--ink);}

  .hero{
    background:var(--panel);
    color:var(--panel-text);
    padding:88px 24px 76px;
    position:relative;
    overflow:hidden;
  }
  .hero-inner{max-width:var(--maxw); margin:0 auto; position:relative; z-index:2;}
  .eyebrow{
    text-transform:uppercase; font-size:12.5px; letter-spacing:.16em;
    color:#9AA3B0; font-weight:600; margin-bottom:22px;
    display:flex; align-items:center; gap:10px;
  }
  .hero h1{font-size:44px; line-height:1.12; font-weight:700; margin:0 0 22px; letter-spacing:-0.01em; max-width:12em;}
  .hero .dek{font-family:'Source Serif 4', serif; font-size:19px; line-height:1.6; color:#C4C9D2; max-width:38em; margin:0 0 30px;}
  .byline{font-family:'Space Grotesk',sans-serif; font-size:13.5px; color:#8B93A0; display:flex; gap:18px; flex-wrap:wrap; align-items:center;}
  .byline strong{color:#D7DAE1; font-weight:600;}

  .hero-graphic{position:absolute; right:-50px; top:50%; transform:translateY(-50%); width:340px; height:340px; opacity:.9; z-index:1;}
  @media (max-width:900px){ .hero-graphic{display:none;} }

  .layout{
    max-width:1180px; margin:0 auto; padding:56px 24px 100px;
    display:grid; grid-template-columns:230px 1fr; gap:56px;
  }
  @media (max-width:980px){ .layout{grid-template-columns:1fr; padding:40px 20px 80px;} .toc{display:none;} }
  .toc{position:sticky; top:76px; align-self:start; font-family:'Space Grotesk',sans-serif; font-size:13px; max-height:calc(100vh - 100px); overflow-y:auto;}
  .toc .toc-label{text-transform:uppercase; letter-spacing:.12em; font-size:11px; color:var(--ink-soft); font-weight:600; margin-bottom:14px;}
  .toc ol{list-style:none; margin:0; padding:0; border-left:1px solid var(--line);}
  .toc a{display:block; padding:6px 0 6px 16px; color:var(--ink-soft); text-decoration:none; border-left:2px solid transparent; margin-left:-1px; transition:color .15s, border-color .15s;}
  .toc a:hover{color:var(--ink); border-left-color:var(--line);}
  .toc a.active{color:var(--ink); border-left-color:var(--blue); font-weight:600;}

  article{max-width:var(--maxw);}
  article h2{font-size:27px; font-weight:700; margin:66px 0 6px; letter-spacing:-0.005em; scroll-margin-top:84px;}
  article h2 .num{color:var(--ink-soft); font-weight:500; margin-right:10px;}
  article h3{font-size:18.5px; font-weight:600; margin:30px 0 10px; scroll-margin-top:84px;}
  article h4{font-size:15px; font-weight:700; margin:22px 0 8px; text-transform:uppercase; letter-spacing:.06em; color:var(--ink-soft);}
  article p{margin:0 0 18px;}
  article ul, article ol{margin:0 0 18px; padding-left:22px;}
  article li{margin-bottom:6px;}
  .section-intro{color:var(--ink-soft); font-size:17px;}
  .kicker{
    display:inline-flex; align-items:center; gap:8px; font-family:'Space Grotesk',sans-serif;
    font-size:12px; font-weight:700; letter-spacing:.08em; text-transform:uppercase;
    color:var(--ink-soft); margin-bottom:2px;
  }

  strong{font-weight:600;}
  code{background:var(--paper-2); border:1px solid var(--line); border-radius:4px; padding:1.5px 6px; font-size:.86em; color:#1F3A93;}

  pre{background:var(--panel); color:var(--panel-text); border-radius:var(--radius); padding:18px 20px; overflow-x:auto; font-size:13.5px; line-height:1.6; margin:22px 0; box-shadow:0 1px 0 rgba(0,0,0,.05);}
  pre code{background:none; border:none; padding:0; color:inherit; font-size:1em;}
  .pre-label{font-family:'Space Grotesk',sans-serif; font-size:11.5px; text-transform:uppercase; letter-spacing:.1em; color:var(--ink-soft); margin:26px 0 -8px;}

  blockquote{margin:22px 0; padding:14px 20px; border-left:3px solid var(--blue); background:var(--blue-bg); border-radius:0 8px 8px 0; font-size:16.5px; color:#1B2A4D;}
  blockquote p{margin:0;}
  blockquote cite{display:block; margin-top:8px; font-family:'Space Grotesk',sans-serif; font-size:12.5px; font-style:normal; color:#3E5CA8;}

  .callout{border:1px solid var(--line); background:#fff; border-radius:var(--radius); padding:22px 24px; margin:26px 0;}
  .callout-title{font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:13px; text-transform:uppercase; letter-spacing:.1em; margin-bottom:10px; display:flex; align-items:center; gap:8px;}
  .callout.green{border-color:#BFDECB; background:var(--green-bg);} .callout.green .callout-title{color:#1F6B47;}
  .callout.amber{border-color:#E7CE9E; background:var(--amber-bg);} .callout.amber .callout-title{color:#8C5E13;}
  .callout.red{border-color:#E9BFC0; background:var(--red-bg);} .callout.red .callout-title{color:#8E2E32;}
  .callout.blue{border-color:#C3D2F5; background:var(--blue-bg);} .callout.blue .callout-title{color:#1F3A93;}

  figure.diagram{margin:28px 0 32px; padding:22px 20px 16px; background:#fff; border:1px solid var(--line); border-radius:var(--radius);}
  figure.diagram svg{width:100%; height:auto; display:block;}
  figure.diagram figcaption{font-family:'Space Grotesk',sans-serif; font-size:13px; color:var(--ink-soft); margin-top:10px; text-align:center;}

  .tablewrap{overflow-x:auto; margin:22px 0;}
  table{border-collapse:collapse; width:100%; font-size:15px;}
  th,td{border:1px solid var(--line); padding:10px 12px; text-align:left; vertical-align:top;}
  th{background:var(--paper-2); font-family:'Space Grotesk',sans-serif; font-weight:600; font-size:13px; text-transform:uppercase; letter-spacing:.04em; color:var(--ink-soft);}
  td.center, th.center{text-align:center;}
  .dot{display:inline-block; width:10px; height:10px; border-radius:50%; margin-right:6px; vertical-align:middle;}
  .dot.r{background:var(--red);} .dot.a{background:var(--amber);} .dot.g{background:var(--green);}

  .chip{display:inline-block; font-size:12px; font-weight:600; padding:3px 10px; border-radius:20px; letter-spacing:.02em;}
  .chip.g{background:var(--green-bg); color:#1F6B47;} .chip.a{background:var(--amber-bg); color:#8C5E13;}
  .chip.r{background:var(--red-bg); color:#8E2E32;} .chip.b{background:var(--blue-bg); color:#1F3A93;}

  .cards{display:grid; grid-template-columns:1fr 1fr; gap:18px; margin:26px 0;}
  @media (max-width:640px){ .cards{grid-template-columns:1fr;} }
  .card{border:1px solid var(--line); border-radius:var(--radius); padding:20px; background:#fff;}
  .card h4{margin:0 0 4px; font-family:'Space Grotesk',sans-serif; font-size:16px; font-weight:700; text-transform:none; letter-spacing:0; color:var(--ink);}
  .card .sub{color:var(--ink-soft); font-size:13.5px; margin-bottom:14px; font-family:'Space Grotesk',sans-serif;}
  .card ul{padding-left:18px; margin:0; font-size:15px;}
  .card li{margin-bottom:6px;}

  /* failure-mode section header badge */
  .modebadge{
    display:inline-flex; align-items:center; justify-content:center;
    width:34px; height:34px; border-radius:8px; background:var(--panel); color:#fff;
    font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:14px; margin-right:12px;
  }
  .mode-h2{display:flex; align-items:center;}

  /* checklist */
  .checklist{list-style:none; margin:22px 0; padding:0; counter-reset:step;}
  .checklist li{
    counter-increment:step; position:relative; padding:14px 18px 14px 54px; margin-bottom:10px;
    background:#fff; border:1px solid var(--line); border-radius:var(--radius); font-size:16px;
  }
  .checklist li::before{
    content:counter(step); position:absolute; left:16px; top:50%; transform:translateY(-50%);
    width:26px; height:26px; border-radius:50%; background:var(--panel); color:#fff;
    font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:13px;
    display:flex; align-items:center; justify-content:center;
  }
  .checklist .label{font-family:'Space Grotesk',sans-serif; font-weight:700; display:block; margin-bottom:2px;}

  .cheat{background:var(--panel); color:var(--panel-text); border-radius:var(--radius); padding:8px; margin:28px 0;}
  .cheat-table{width:100%; border-collapse:collapse; font-size:13.5px;}
  .cheat-table td, .cheat-table th{border:1px solid #333A45; padding:11px 13px; text-align:left;}
  .cheat-table th{background:var(--panel-2); color:#B9C0CC; font-weight:600;}

  hr.rule{border:none; border-top:1px solid var(--line); margin:56px 0;}

  footer{max-width:var(--maxw); margin:0 auto; padding:0 24px 90px; font-size:14px; color:var(--ink-soft);}
  footer a{color:var(--ink-soft); text-decoration:underline;}
  .footer-refs{list-style:none; padding:0; margin:14px 0 0;}
  .footer-refs li{margin-bottom:6px;}
  .ref-group{margin-top:22px;}
  .ref-group .group-title{font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:12.5px; text-transform:uppercase; letter-spacing:.08em; color:var(--ink); margin-bottom:8px;}

  ::selection{background:#CFE0FF;}
  @media (prefers-reduced-motion: reduce){ html{scroll-behavior:auto;} }
</style>
</head>
<body>

<nav class="topnav">
  <a class="brand" href="index.html">
    <span class="signal"><span class="r"></span><span class="a"></span><span class="g"></span></span>
    System Notes
  </a>
  <a class="back" href="index.html">← All guides</a>
</nav>

<header class="hero">
  <div class="hero-inner">
    <div class="eyebrow">
      <span class="signal"><span class="r"></span><span class="a"></span><span class="g"></span></span>
      Backend&nbsp;/&nbsp;System Design Interviews
    </div>
    <h1>Backend Under Load: Eight Failure Modes</h1>
    <p class="dek">"Your p99 just went from 50ms to 5000ms during a traffic spike — walk me through it." Here are the eight usual suspects behind that sentence: what each one is, why it happens, how you'd catch it in metrics, how to fix it, and how they chain into each other in a real incident.</p>
    <div class="byline">
      <span><strong>Read time</strong> ~22 min</span>
      <span><strong>Level</strong> 3+ YOE backend / system design</span>
      <span><strong>Format</strong> Interview field guide</span>
    </div>
  </div>
  <svg class="hero-graphic" viewBox="0 0 340 340" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
    <circle cx="170" cy="170" r="4" fill="#E9EBEE"/>
    <g stroke="#2A3040" stroke-width="1" fill="none">
      <path d="M170,170 L170,40"/><path d="M170,170 L280,95"/><path d="M170,170 L300,200"/>
      <path d="M170,170 L230,290"/><path d="M170,170 L100,290"/><path d="M170,170 L50,200"/><path d="M170,170 L70,90"/>
    </g>
    <circle cx="170" cy="40" r="8" fill="#C7454A"/>
    <circle cx="280" cy="95" r="8" fill="#C8862B"/>
    <circle cx="300" cy="200" r="8" fill="#C7454A"/>
    <circle cx="230" cy="290" r="8" fill="#2F8F63"/>
    <circle cx="100" cy="290" r="8" fill="#C8862B"/>
    <circle cx="50" cy="200" r="8" fill="#2F8F63"/>
    <circle cx="70" cy="90" r="8" fill="#C8862B"/>
  </svg>
</header>

<div class="layout">
  <nav class="toc" id="toc">
    <div class="toc-label">On this page</div>
    <ol>
      <li><a href="#pool">1. Connection pool saturation</a></li>
      <li><a href="#cache-storm">2. Cache miss storm</a></li>
      <li><a href="#thundering-herd">3. Thundering herd</a></li>
      <li><a href="#lock-contention">4. Lock contention</a></li>
      <li><a href="#hot-partitions">5. Hot partitions</a></li>
      <li><a href="#slow-queries">6. Slow queries</a></li>
      <li><a href="#network">7. Network bottlenecks</a></li>
      <li><a href="#gc">8. GC pauses</a></li>
      <li><a href="#checklist">9. Diagnostic checklist</a></li>
      <li><a href="#cascade">10. How they cascade</a></li>
      <li><a href="#references">11. References</a></li>
    </ol>
  </nav>

  <article>

    <p class="section-intro">Interviewers at this level rarely want a fresh whiteboard architecture. They want to know what you do when the architecture you already drew starts to buckle — because in production, every one of these eight failure modes shows up sooner or later, usually more than one at a time. The skill being tested is <strong>symptom → correlated metric → root cause → fix</strong>, not trivia recall.</p>

    <div class="callout blue">
      <div class="callout-title"><span class="dot r"></span><span class="dot a"></span><span class="dot g"></span>&nbsp;How to read this guide</div>
      Each failure mode below follows the same shape: what it is, why it happens, how you'd catch it in metrics or logs, how to fix or prevent it, and a scenario you can talk through out loud. Red marks an active failure signal, amber a degraded/borderline one, green a healthy or fixed state — the same color language carries through every diagram.
    </div>

    <h2 id="pool" class="mode-h2"><span class="modebadge">01</span>Connection pool saturation</h2>
    <p>Your application doesn't talk to the database directly on every request. It borrows a connection from a fixed-size pool (HikariCP in Java is the canonical example), uses it, and returns it. When every connection is checked out and requests keep arriving, new requests queue behind the pool. If they wait long enough, they time out.</p>

    <h4>Why it happens</h4>
    <ul>
      <li>Pool sized for average load, not actual peak concurrency.</li>
      <li>Connections held too long — long transactions, leaked connections that never return, N+1 queries inside a single transaction.</li>
      <li>A downstream slowdown (slow queries, a stalled dependency) means each connection is held longer than usual — the pool's <em>effective</em> throughput drops even though its size hasn't changed.</li>
      <li><strong>Fleet math is the real trap:</strong> 50 pods × a pool size of 20 offers 1,000 simultaneous connections to a database that may handle a few hundred well. The pool protects a single instance — nothing protects the database as a whole unless you design for it.</li>
    </ul>

    <figure class="diagram">
      <svg viewBox="0 0 700 240" xmlns="http://www.w3.org/2000/svg" role="img" aria-labelledby="pool-title">
        <title id="pool-title">Connection pool exhaustion under a traffic spike</title>
        <defs>
          <marker id="a1" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" fill="#565F6B"/></marker>
        </defs>
        <text x="20" y="26" font-family="Space Grotesk" font-size="12" font-weight="600" fill="var(--ink-soft)" style="fill:#565F6B">incoming requests</text>
        <rect x="20" y="36" width="30" height="18" rx="3" fill="#DFE6F9"/>
        <rect x="55" y="36" width="30" height="18" rx="3" fill="#DFE6F9"/>
        <rect x="90" y="36" width="30" height="18" rx="3" fill="#DFE6F9"/>
        <rect x="125" y="36" width="30" height="18" rx="3" fill="#DFE6F9"/>
        <rect x="160" y="36" width="30" height="18" rx="3" fill="#F6E9D3"/>
        <rect x="195" y="36" width="30" height="18" rx="3" fill="#F6E9D3"/>
        <path d="M225,45 L260,45" stroke="#565F6B" stroke-width="1.5" marker-end="url(#a1)"/>

        <rect x="260" y="16" width="200" height="60" rx="8" fill="#fff" stroke="#181C22" stroke-width="1.5"/>
        <text x="360" y="40" text-anchor="middle" font-family="Space Grotesk" font-size="13" font-weight="700" fill="#181C22">Pool (size = 20)</text>
        <text x="360" y="58" text-anchor="middle" font-family="IBM Plex Mono" font-size="11" fill="#565F6B">20 / 20 checked out</text>

        <path d="M360,76 L360,110" stroke="#565F6B" stroke-width="1.5" marker-end="url(#a1)"/>
        <rect x="260" y="112" width="200" height="46" rx="8" fill="#F6E9D3" stroke="#C8862B" stroke-width="1.5"/>
        <text x="360" y="140" text-anchor="middle" font-family="Space Grotesk" font-size="12.5" font-weight="700" fill="#8C5E13">queue growing</text>

        <path d="M460,135 L560,105" stroke="#2F8F63" stroke-width="1.5" marker-end="url(#a1)"/>
        <rect x="560" y="76" width="120" height="56" rx="8" fill="#DDEFE4" stroke="#2F8F63" stroke-width="1.5"/>
        <text x="620" y="98" text-anchor="middle" font-family="Space Grotesk" font-size="12" font-weight="700" fill="#1F6B47">DB</text>
        <text x="620" y="115" text-anchor="middle" font-family="Space Grotesk" font-size="10.5" fill="#1F6B47">not the bottleneck</text>

        <path d="M460,155 L560,190" stroke="#C7454A" stroke-width="1.5" marker-end="url(#a1)"/>
        <rect x="560" y="170" width="120" height="56" rx="8" fill="#F7E4E3" stroke="#C7454A" stroke-width="1.5"/>
        <text x="620" y="192" text-anchor="middle" font-family="Space Grotesk" font-size="12" font-weight="700" fill="#8E2E32">Timeout</text>
        <text x="620" y="210" text-anchor="middle" font-family="Space Grotesk" font-size="10.5" fill="#8E2E32">after connectionTimeout</text>
      </svg>
      <figcaption>The database is often fine — it's the pool in front of it that runs out of seats.</figcaption>
    </figure>

    <h4>Diagnosis</h4>
    <ul>
      <li>Pool metrics: active connections ≈ max pool size, plus a growing "waiting for connection" queue.</li>
      <li>Symptom: <code>SQLTransientConnectionException: Connection is not available, request timed out</code> (HikariCP), or the equivalent driver timeout elsewhere.</li>
      <li>Correlate pool exhaustion with query latency, not just request volume — often the pool wasn't undersized, the queries got slow.</li>
    </ul>

    <h4>Fix / prevention</h4>
    <ul>
      <li><strong>Right-size the pool</strong> using HikariCP's starting-point formula, then tune with real load tests: <code>connections ≈ (core_count × 2) + effective_spindle_count</code>. Bigger isn't always better — excess connections add context-switching and contention on the DB itself.</li>
      <li><strong>Fail fast:</strong> set an aggressive <code>connectionTimeout</code> (2–3s) so requests error out visibly instead of queuing silently.</li>
      <li><strong>Cap total connections across the fleet</strong>, not per instance — think in terms of a database connection budget split across replicas, batch jobs, and admin tooling.</li>
      <li>Put a multiplexer/proxy (PgBouncer, ProxySQL, Oracle CMAN) between app and DB to decouple "connections the app thinks it opened" from "connections the DB actually maintains." PgBouncer's <em>transaction</em> pooling mode in particular can serve hundreds of app connections from a much smaller real pool.</li>
      <li>Turn on leak detection (HikariCP's <code>leak-detection-threshold</code>) to catch code paths that check out a connection and never return it.</li>
      <li>Watch for autoscaling "connection storms": 50 new pods opening a full pool at boot can spike DB CPU in seconds. Stagger warm-up — start with <code>minimum-idle: 1</code> and grow lazily.</li>
    </ul>

    <div class="pre-label">HikariCP — safer defaults for a fleet</div>
    <pre><code>spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=1
spring.datasource.hikari.connection-timeout=2500
spring.datasource.hikari.leak-detection-threshold=30000
spring.datasource.hikari.validation-timeout=1000</code></pre>

    <div class="callout amber">
      <div class="callout-title"><span class="dot a"></span>Scenario to reason through</div>
      A Spring Boot service behind an HPA. Traffic doubles during a flash sale; Kubernetes scales pods 20 → 80. The database itself falls over — not the app. <strong>This is a connection storm:</strong> 80 pods × pool size 10 = 800 sudden new connections. The fix isn't "raise the pool size" — it's capping total DB connections fleet-wide and warming pools gradually.
    </div>

    <h2 id="cache-storm" class="mode-h2"><span class="modebadge">02</span>Cache miss storm</h2>
    <p>A cache exists to shield the database from repeated reads of the same data. A cache miss storm happens when a large chunk of the cache becomes invalid or unavailable at once — a cache node restarts, a deploy flushes it, or many keys share a TTL and expire together — and a flood of requests that would normally hit the cache all fall through to the database simultaneously.</p>

    <h4>Why it happens</h4>
    <ul>
      <li><strong>Synchronized TTLs:</strong> many keys cached at the same moment with the same expiry all die together.</li>
      <li><strong>Cold cache after deploy or restart:</strong> a new cache instance, or a cluster failover, starts empty.</li>
      <li><strong>Eviction under memory pressure:</strong> LRU eviction removes hot keys exactly when you need them most.</li>
    </ul>

    <figure class="diagram">
      <svg viewBox="0 0 700 200" xmlns="http://www.w3.org/2000/svg" role="img" aria-labelledby="storm-title">
        <title id="storm-title">Mass cache expiry floods the database</title>
        <defs><marker id="a2" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" fill="#565F6B"/></marker></defs>
        <rect x="20" y="70" width="180" height="60" rx="8" fill="#F6E9D3" stroke="#C8862B" stroke-width="1.5"/>
        <text x="110" y="94" text-anchor="middle" font-family="Space Grotesk" font-size="12.5" font-weight="700" fill="#8C5E13">Cache cluster</text>
        <text x="110" y="112" text-anchor="middle" font-family="Space Grotesk" font-size="11" fill="#8C5E13">restarts / mass TTL expiry</text>

        <path d="M200,100 L260,100" stroke="#565F6B" stroke-width="1.5" marker-end="url(#a2)"/>
        <g>
          <rect x="260" y="30" width="60" height="26" rx="4" fill="#F7E4E3" stroke="#C7454A"/>
          <rect x="260" y="65" width="60" height="26" rx="4" fill="#F7E4E3" stroke="#C7454A"/>
          <rect x="260" y="100" width="60" height="26" rx="4" fill="#F7E4E3" stroke="#C7454A"/>
          <rect x="260" y="135" width="60" height="26" rx="4" fill="#F7E4E3" stroke="#C7454A"/>
          <text x="290" y="47" text-anchor="middle" font-family="IBM Plex Mono" font-size="10" fill="#8E2E32">MISS</text>
          <text x="290" y="82" text-anchor="middle" font-family="IBM Plex Mono" font-size="10" fill="#8E2E32">MISS</text>
          <text x="290" y="117" text-anchor="middle" font-family="IBM Plex Mono" font-size="10" fill="#8E2E32">MISS</text>
          <text x="290" y="152" text-anchor="middle" font-family="IBM Plex Mono" font-size="10" fill="#8E2E32">MISS</text>
        </g>
        <path d="M320,43 L440,90" stroke="#C7454A" stroke-width="1.2" marker-end="url(#a2)"/>
        <path d="M320,78 L440,95" stroke="#C7454A" stroke-width="1.2" marker-end="url(#a2)"/>
        <path d="M320,113 L440,100" stroke="#C7454A" stroke-width="1.2" marker-end="url(#a2)"/>
        <path d="M320,148 L440,105" stroke="#C7454A" stroke-width="1.2" marker-end="url(#a2)"/>

        <rect x="440" y="65" width="130" height="65" rx="8" fill="#fff" stroke="#181C22" stroke-width="1.5"/>
        <text x="505" y="93" text-anchor="middle" font-family="Space Grotesk" font-size="13" font-weight="700" fill="#181C22">Database</text>
        <text x="505" y="111" text-anchor="middle" font-family="Space Grotesk" font-size="11" fill="#565F6B">every miss lands here</text>

        <path d="M570,97 L640,97" stroke="#C7454A" stroke-width="1.5" marker-end="url(#a2)"/>
        <rect x="600" y="70" width="80" height="55" rx="8" fill="#F7E4E3" stroke="#C7454A" stroke-width="1.5"/>
        <text x="640" y="93" text-anchor="middle" font-family="Space Grotesk" font-size="11.5" font-weight="700" fill="#8E2E32">Timeouts</text>
        <text x="640" y="110" text-anchor="middle" font-family="Space Grotesk" font-size="10" fill="#8E2E32">high latency</text>
      </svg>
      <figcaption>Broad and multi-key — the whole cache goes cold at once, not just one popular entry.</figcaption>
    </figure>

    <h4>Diagnosis</h4>
    <ul>
      <li>Sudden, correlated spike in miss rate across <em>many</em> keys (contrast with Thundering Herd below, which is one key).</li>
      <li>DB read QPS spikes in lockstep with a cache hit-rate drop.</li>
      <li>Usually visible right after a deploy, a cache cluster resize, or a node failover.</li>
    </ul>

    <h4>Fix / prevention</h4>
    <ul>
      <li><strong>Jittered TTLs</strong> — add randomness to expiry: <code>TTL = base ± random(0, jitter)</code>, so keys don't all die at the same instant.</li>
      <li><strong>Cache warming</strong> before cutting traffic over to a new node or cluster, especially post-deploy.</li>
      <li><strong>Graceful degradation:</strong> serve slightly stale data (stale-while-revalidate) instead of forcing every reader to the DB.</li>
      <li><strong>Multi-tier caching:</strong> local in-process cache + shared Redis/Memcached, so a shared-cache blip doesn't send 100% of traffic straight to the DB.</li>
      <li><strong>Read-through with request coalescing</strong> (see Thundering Herd) so even a full flush produces one DB query per key, not one per concurrent request.</li>
    </ul>

    <h2 id="thundering-herd" class="mode-h2"><span class="modebadge">03</span>Thundering herd</h2>
    <p>An especially vicious, narrower case of a cache miss: <strong>one single hot key</strong> expires, and every one of its many concurrent readers independently notices the miss and hits the database at the same instant to regenerate it. The name borrows from the general OS-level phenomenon where many threads blocked on the same event all wake up simultaneously, but only one can actually proceed — the rest collide and go back to sleep, wasting the cycles it took to wake them.</p>

    <p>A widely-used mental model: a product recommendation cache key expires at scale, and tens of thousands of concurrent requests hit the primary database within seconds — CPU spikes toward saturation, checkout pages slow to a crawl, and the root cause is "just" one expired cache key.</p>

    <figure class="diagram">
      <svg viewBox="0 0 700 300" xmlns="http://www.w3.org/2000/svg" role="img" aria-labelledby="herd-title">
        <title id="herd-title">Thundering herd, with and without request coalescing</title>
        <defs><marker id="a3" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" fill="#565F6B"/></marker></defs>

        <text x="20" y="24" font-family="Space Grotesk" font-size="12" font-weight="700" fill="#8E2E32">WITHOUT COALESCING</text>
        <rect x="20" y="36" width="150" height="40" rx="6" fill="#F7E4E3" stroke="#C7454A"/>
        <text x="95" y="60" text-anchor="middle" font-family="Space Grotesk" font-size="11.5" fill="#8E2E32">hot key expires</text>
        <path d="M170,56 L230,40" stroke="#565F6B" stroke-width="1.3" marker-end="url(#a3)"/>
        <path d="M170,56 L230,72" stroke="#565F6B" stroke-width="1.3" marker-end="url(#a3)"/>
        <text x="250" y="35" font-family="IBM Plex Mono" font-size="10" fill="#565F6B">req 1</text>
        <text x="250" y="80" font-family="IBM Plex Mono" font-size="10" fill="#565F6B">req 2..N</text>
        <path d="M290,38 L400,58" stroke="#C7454A" stroke-width="1.3" marker-end="url(#a3)"/>
        <path d="M290,75 L400,60" stroke="#C7454A" stroke-width="1.3" marker-end="url(#a3)"/>
        <rect x="400" y="36" width="150" height="46" rx="6" fill="#F7E4E3" stroke="#C7454A" stroke-width="1.5"/>
        <text x="475" y="55" text-anchor="middle" font-family="Space Grotesk" font-size="11.5" font-weight="700" fill="#8E2E32">Database</text>
        <text x="475" y="72" text-anchor="middle" font-family="Space Grotesk" font-size="10" fill="#8E2E32">N identical queries at once</text>

        <line x1="10" y1="105" x2="690" y2="105" stroke="#D8DBD1"/>

        <text x="20" y="130" font-family="Space Grotesk" font-size="12" font-weight="700" fill="#1F6B47">WITH SINGLE-FLIGHTING</text>
        <rect x="20" y="142" width="150" height="40" rx="6" fill="#DDEFE4" stroke="#2F8F63"/>
        <text x="95" y="166" text-anchor="middle" font-family="Space Grotesk" font-size="11.5" fill="#1F6B47">key expires</text>
        <path d="M170,155 L230,145" stroke="#565F6B" stroke-width="1.3" marker-end="url(#a3)"/>
        <path d="M170,170 L230,180" stroke="#565F6B" stroke-width="1.3" marker-end="url(#a3)"/>
        <rect x="230" y="130" width="150" height="34" rx="6" fill="#fff" stroke="#181C22"/>
        <text x="305" y="151" text-anchor="middle" font-family="Space Grotesk" font-size="11" fill="#181C22">req 1: acquires lock</text>
        <rect x="230" y="170" width="150" height="34" rx="6" fill="#fff" stroke="#181C22"/>
        <text x="305" y="191" text-anchor="middle" font-family="Space Grotesk" font-size="11" fill="#181C22">req 2..N: see lock, wait</text>

        <path d="M380,147 L470,147" stroke="#2F8F63" stroke-width="1.3" marker-end="url(#a3)"/>
        <rect x="470" y="130" width="120" height="34" rx="6" fill="#DDEFE4" stroke="#2F8F63" stroke-width="1.5"/>
        <text x="530" y="151" text-anchor="middle" font-family="Space Grotesk" font-size="11" font-weight="700" fill="#1F6B47">1 query only</text>

        <path d="M530,164 L530,215" stroke="#2F8F63" stroke-width="1.3" marker-end="url(#a3)"/>
        <path d="M470,187 L400,220" stroke="#2F8F63" stroke-width="1.3" marker-end="url(#a3)"/>
        <rect x="380" y="222" width="220" height="40" rx="6" fill="#DDEFE4" stroke="#2F8F63" stroke-width="1.5"/>
        <text x="490" y="247" text-anchor="middle" font-family="Space Grotesk" font-size="11.5" font-weight="700" fill="#1F6B47">cache refreshed once — everyone reads it</text>
      </svg>
      <figcaption>Same event, two outcomes: N duplicate queries, or exactly one — the only difference is whether a lock elects a single rebuilder.</figcaption>
    </figure>

    <h4>Diagnosis</h4>
    <ul>
      <li>One specific key or query pattern dominates the miss spike (versus the broad, multi-key pattern of a cache miss storm).</li>
      <li>DB slow query log shows the <em>exact same query</em> running dozens or hundreds of times concurrently.</li>
      <li>Correlates precisely with the TTL of one hot object.</li>
    </ul>

    <h4>Fix / prevention</h4>
    <ul>
      <li><strong>Request coalescing / single-flighting:</strong> on a miss, the first request acquires a lock and regenerates the cache; every other concurrent request for the same key waits briefly, then reads the now-fresh entry instead of also hitting the DB. The design question becomes "how do I elect a single leader to rebuild this key," not "what happens on a miss."</li>
      <li><strong>Probabilistic early expiration:</strong> refresh the key before it actually expires, with rising probability as expiry approaches — regeneration happens off the critical path, with no hard cliff.</li>
      <li><strong>Never let TTL be the only expiry mechanism for hot keys</strong> — actively refresh them in the background via cron/worker so they never truly miss.</li>
      <li><strong>Locking with a short grace period:</strong> hold a mutex per key during regeneration; other readers serve stale data for a few hundred ms rather than blocking on the DB.</li>
    </ul>

    <div class="pre-label">Single-flighting, in pseudocode</div>
    <pre><code>value = cache.get(key)
if value is not None:
    return value

if acquire_lock(key, ttl=500ms):        # only one caller wins this
    try:
        value = db.query(...)
        cache.set(key, value, ttl=jittered(base_ttl))
        return value
    finally:
        release_lock(key)
else:
    # someone else is already rebuilding — wait briefly, then re-read
    sleep(50ms)
    return cache.get(key) or stale_cache.get(key)</code></pre>

    <h2 id="lock-contention" class="mode-h2"><span class="modebadge">04</span>Lock contention</h2>
    <p>Multiple threads or processes need exclusive access to the same resource — a row, a data structure, a critical section — and end up serializing behind a lock. As concurrency rises, more time is spent waiting for the lock than doing useful work.</p>

    <h4>Why it happens</h4>
    <ul>
      <li>Coarse-grained locks — locking an entire table or object when only a small part needs protection.</li>
      <li>Long-held locks — a transaction holding a row lock while doing slow, unrelated work like an external API call.</li>
      <li>High contention on a small number of hot rows — everyone incrementing the same counter, many transactions updating the same inventory record.</li>
      <li>In application code: synchronized blocks or mutexes guarding shared in-memory state.</li>
    </ul>

    <figure class="diagram">
      <svg viewBox="0 0 700 220" xmlns="http://www.w3.org/2000/svg" role="img" aria-labelledby="lock-title">
        <title id="lock-title">Threads queued behind one lock</title>
        <circle cx="350" cy="110" r="34" fill="#F7E4E3" stroke="#C7454A" stroke-width="2"/>
        <text x="350" y="106" text-anchor="middle" font-family="Space Grotesk" font-size="11" font-weight="700" fill="#8E2E32">Lock on</text>
        <text x="350" y="120" text-anchor="middle" font-family="Space Grotesk" font-size="11" font-weight="700" fill="#8E2E32">Row X</text>

        <g font-family="Space Grotesk" font-size="11.5" fill="#181C22">
          <rect x="30" y="20" width="120" height="34" rx="6" fill="#DDEFE4" stroke="#2F8F63"/>
          <text x="90" y="42" text-anchor="middle" fill="#1F6B47" font-weight="700">Thread 0 — running</text>
          <path d="M150,37 L316,95" stroke="#2F8F63" stroke-width="1.5"/>

          <rect x="30" y="70" width="120" height="34" rx="6" fill="#F6E9D3" stroke="#C8862B"/>
          <text x="90" y="92" text-anchor="middle" fill="#8C5E13">Thread 1 — waits</text>
          <path d="M150,87 L316,105" stroke="#C8862B" stroke-width="1.5" stroke-dasharray="4 3"/>

          <rect x="30" y="120" width="120" height="34" rx="6" fill="#F6E9D3" stroke="#C8862B"/>
          <text x="90" y="142" text-anchor="middle" fill="#8C5E13">Thread 2 — waits</text>
          <path d="M150,137 L316,115" stroke="#C8862B" stroke-width="1.5" stroke-dasharray="4 3"/>

          <rect x="30" y="170" width="120" height="34" rx="6" fill="#F6E9D3" stroke="#C8862B"/>
          <text x="90" y="192" text-anchor="middle" fill="#8C5E13">Thread 3 — waits</text>
          <path d="M150,187 L316,125" stroke="#C8862B" stroke-width="1.5" stroke-dasharray="4 3"/>
        </g>
        <text x="520" y="108" font-family="Space Grotesk" font-size="12" fill="#565F6B">throughput ≈ 1 thread's</text>
        <text x="520" y="124" font-family="Space Grotesk" font-size="12" fill="#565F6B">worth of work, regardless</text>
        <text x="520" y="140" font-family="Space Grotesk" font-size="12" fill="#565F6B">of how many threads exist</text>
      </svg>
      <figcaption>Classic tell: CPU can look low even though latency is high — the bottleneck is waiting, not computing.</figcaption>
    </figure>

    <h4>Diagnosis</h4>
    <ul>
      <li>Thread dumps show many threads <code>BLOCKED</code> waiting on the same monitor/lock.</li>
      <li>Database: <code>SHOW ENGINE INNODB STATUS</code> (MySQL) or <code>pg_locks</code> (Postgres) reveals long lock-wait chains and deadlock victims.</li>
      <li>CPU utilization can look low even though latency is high — a classic tell that the bottleneck is waiting, not computing.</li>
      <li>Throughput plateaus or drops even as you add more threads or instances — Amdahl's Law in action, the serialized portion dominates.</li>
    </ul>

    <h4>Fix / prevention</h4>
    <ul>
      <li><strong>Reduce lock scope and hold time:</strong> do slow work (I/O, external calls) outside the critical section.</li>
      <li><strong>Move from pessimistic to optimistic concurrency control</strong> where contention is high but conflicts are actually rare — versioned rows / compare-and-swap instead of <code>SELECT ... FOR UPDATE</code>.</li>
      <li><strong>Shard the contended resource:</strong> instead of one counter row, use N counter shards and sum them (same principle as hot partitions, below).</li>
      <li><strong>Use appropriate isolation levels:</strong> don't default to the strictest level everywhere — know your actual consistency requirement.</li>
      <li><strong>Queue-based serialization:</strong> if a resource genuinely must update one-at-a-time, make that explicit via a queue rather than many threads fighting over a lock.</li>
    </ul>

    <h2 id="hot-partitions" class="mode-h2"><span class="modebadge">05</span>Hot partitions</h2>
    <p>In a sharded or partitioned data store (DynamoDB, Cassandra, sharded Postgres/MySQL), data spreads across nodes by hashing a partition key. A hot partition happens when a disproportionate share of traffic lands on one partition — because one key, or a small set of keys, is far more popular than the rest — even though the cluster as a whole has spare capacity.</p>

    <h4>Why it happens</h4>
    <ul>
      <li><strong>Low-cardinality keys:</strong> partitioning by something like <code>status = "ACTIVE"</code> or a date-only value concentrates huge volume on very few partition values.</li>
      <li><strong>Celebrity effect:</strong> one user, one product, one leaderboard entry gets orders of magnitude more reads/writes than everyone else.</li>
      <li><strong>Time-series data:</strong> keying by <code>deviceId</code> or a fixed date bucket means all of "today's" writes land on the same physical partition, since time-ordered data inherently targets one logical key repeatedly.</li>
      <li><strong>Per-partition limits are hard limits</strong> — in DynamoDB, a single partition tops out around 3,000 RCU / 1,000 WCU <em>regardless of how much total capacity the table has provisioned</em>, so a hot partition throttles even when the table "looks" underutilized.</li>
    </ul>

    <figure class="diagram">
      <svg viewBox="0 0 700 210" xmlns="http://www.w3.org/2000/svg" role="img" aria-labelledby="hot-title">
        <title id="hot-title">One partition absorbing most of the traffic</title>
        <defs><marker id="a4" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" fill="#565F6B"/></marker></defs>
        <text x="20" y="20" font-family="Space Grotesk" font-size="12" fill="#565F6B">Table: 10,000 WCU total, 1,000 WCU per partition</text>

        <rect x="20" y="40" width="140" height="120" rx="6" fill="#fff" stroke="#181C22"/>
        <text x="90" y="62" text-anchor="middle" font-family="Space Grotesk" font-size="11.5" fill="#181C22">Partition 1</text>
        <rect x="35" y="75" width="110" height="65" fill="#DDEFE4"/>
        <text x="90" y="112" text-anchor="middle" font-family="IBM Plex Mono" font-size="10.5" fill="#1F6B47">~200 WCU</text>

        <rect x="180" y="40" width="140" height="120" rx="6" fill="#fff" stroke="#181C22"/>
        <text x="250" y="62" text-anchor="middle" font-family="Space Grotesk" font-size="11.5" fill="#181C22">Partition 2</text>
        <rect x="195" y="80" width="110" height="60" fill="#DDEFE4"/>
        <text x="250" y="115" text-anchor="middle" font-family="IBM Plex Mono" font-size="10.5" fill="#1F6B47">~180 WCU</text>

        <rect x="340" y="40" width="160" height="120" rx="6" fill="#F7E4E3" stroke="#C7454A" stroke-width="2"/>
        <text x="420" y="62" text-anchor="middle" font-family="Space Grotesk" font-size="12" font-weight="700" fill="#8E2E32">Partition 3 (HOT)</text>
        <rect x="352" y="52" width="136" height="98" fill="#F0B3B4"/>
        <text x="420" y="105" text-anchor="middle" font-family="IBM Plex Mono" font-size="11" font-weight="700" fill="#7A2226">demand: 5,000 WCU</text>
        <text x="420" y="122" text-anchor="middle" font-family="IBM Plex Mono" font-size="10" fill="#7A2226">limit: 1,000 WCU</text>

        <rect x="520" y="40" width="140" height="120" rx="6" fill="#fff" stroke="#181C22"/>
        <text x="590" y="62" text-anchor="middle" font-family="Space Grotesk" font-size="11.5" fill="#181C22">Partition 4</text>
        <rect x="535" y="90" width="110" height="50" fill="#DDEFE4"/>
        <text x="590" y="120" text-anchor="middle" font-family="IBM Plex Mono" font-size="10.5" fill="#1F6B47">~150 WCU</text>

        <path d="M420,20 L420,38" stroke="#C7454A" stroke-width="1.5" marker-end="url(#a4)"/>
        <text x="420" y="14" text-anchor="middle" font-family="Space Grotesk" font-size="11" fill="#8E2E32">90% of client traffic targets one key</text>
      </svg>
      <figcaption>The table looks underutilized in aggregate — the throttling only shows up per-partition.</figcaption>
    </figure>

    <h4>Diagnosis</h4>
    <ul>
      <li>Throttling errors (e.g. <code>ProvisionedThroughputExceededException</code>) even though aggregate table/cluster metrics show spare capacity.</li>
      <li>Per-key traffic visibility tooling (CloudWatch Contributor Insights for DynamoDB, or equivalent) to identify which keys are hot before fixing anything blindly.</li>
    </ul>

    <h4>Fix / prevention</h4>
    <ul>
      <li><strong>Write sharding:</strong> append a random or hashed suffix to the hot key (<code>user#123</code> → <code>user#123#shard3</code>) so writes spread across N physical partitions; fan out reads across all shards and aggregate.</li>
      <li><strong>Composite keys with higher cardinality:</strong> combine a low-cardinality attribute with a high-cardinality one — <code>"ACTIVE#customer123"</code> instead of just <code>"ACTIVE"</code>.</li>
      <li><strong>Shard selectively:</strong> you don't need to shard the whole keyspace — only the specific hot entities (e.g. the top 100 leaderboard users), leaving the long tail on a simple single-key design.</li>
      <li><strong>Cache in front of the hot partition</strong> (DAX/Redis) so the partition itself only sees a fraction of true read volume.</li>
      <li><strong>Time-bucketing for time-series:</strong> roll into new partition keys per time window (per hour/day) instead of one key growing forever.</li>
      <li>Built-in mechanisms like DynamoDB Adaptive Capacity help automatically, but take several minutes to kick in — not fast enough to save you from a sudden spike. Proactive key design still matters.</li>
    </ul>

    <div class="pre-label">Write-sharded key, read-side fan-in</div>
    <pre><code>// write: spread across N shards
shard = random(0, N-1)
put_item(pk=f"user#{userId}#shard{shard}", ...)

// read: fan out and merge
results = [get_item(pk=f"user#{userId}#shard{i}") for i in range(N)]
merged = merge(results)</code></pre>

    <h2 id="slow-queries" class="mode-h2"><span class="modebadge">06</span>Slow queries</h2>
    <p>The most direct root cause on this list — a query that takes far longer than it should, tying up connections, locks, and CPU/IO on the database, and cascading into every other failure mode above.</p>

    <h4>Why it happens</h4>
    <ul>
      <li>Missing or unused indexes, causing full table scans.</li>
      <li>N+1 query patterns from ORMs — one query for a list, then one query per item for related data.</li>
      <li>Poor query plans from stale statistics.</li>
      <li>Large, unbounded result sets — <code>SELECT *</code> with no pagination or limit.</li>
      <li>Lock waits disguised as slow queries — the query itself is fast, but it's queued behind another transaction's lock.</li>
      <li>Open-session-in-view (common in Spring) keeping a DB transaction open for the entire HTTP request lifecycle, including slow downstream calls — so otherwise-fast queries hold connections far longer than necessary.</li>
    </ul>

    <h4>Diagnosis</h4>
    <ul>
      <li>Slow query logs — <code>slow_query_log</code> in MySQL, <code>log_min_duration_statement</code> in Postgres.</li>
      <li><code>EXPLAIN ANALYZE</code> to check for sequential scans where an index scan was expected.</li>
      <li>APM traces showing time spent in DB calls vs. application logic.</li>
      <li>Correlate a rise in average query duration with the rest of the cascade — very often the actual root cause behind "pool exhausted" or "lock wait timeouts spiked."</li>
    </ul>

    <h4>Fix / prevention</h4>
    <ul>
      <li>Add/verify indexes matching actual query predicates and sort orders.</li>
      <li>Fix N+1 patterns with eager loading or batched fetches.</li>
      <li>Set query timeouts so one bad query can't monopolize a connection indefinitely.</li>
      <li>Paginate everything; never return an unbounded result set.</li>
      <li>Disable open-session-in-view where it's not needed; keep transactions as short as possible.</li>
      <li>Use read replicas to offload read-heavy queries away from the primary.</li>
    </ul>

    <h2 id="network" class="mode-h2"><span class="modebadge">07</span>Network bottlenecks</h2>
    <p>Latency or throughput limits introduced by the network path between services, not the services themselves: bandwidth saturation, DNS resolution delays, TLS handshake overhead, cross-region/cross-AZ hops, load balancer or proxy limits.</p>

    <h4>Why it happens</h4>
    <ul>
      <li>Chatty microservices making many small synchronous calls instead of batching.</li>
      <li>Cross-region calls adding tens to hundreds of ms of unavoidable latency per hop.</li>
      <li>Load balancer health checks or connection limits misconfigured — healthy instances marked unhealthy, or connections dropped.</li>
      <li>NIC/bandwidth saturation on a single host under heavy fan-out — broadcasting to many downstream services at once.</li>
      <li>DNS caching issues causing repeated, slow lookups under load.</li>
    </ul>

    <figure class="diagram">
      <svg viewBox="0 0 700 190" xmlns="http://www.w3.org/2000/svg" role="img" aria-labelledby="net-title">
        <title id="net-title">Every hop adds latency; one bad hop can eject healthy instances</title>
        <defs><marker id="a5" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" fill="#565F6B"/></marker></defs>
        <g font-family="Space Grotesk" font-size="11.5">
          <rect x="10" y="70" width="90" height="44" rx="6" fill="#fff" stroke="#181C22"/>
          <text x="55" y="96" text-anchor="middle" fill="#181C22">Client</text>
          <path d="M100,92 L150,92" stroke="#565F6B" stroke-width="1.4" marker-end="url(#a5)"/>
          <text x="125" y="80" text-anchor="middle" font-size="9.5" fill="#565F6B">DNS</text>

          <rect x="150" y="70" width="90" height="44" rx="6" fill="#fff" stroke="#181C22"/>
          <text x="195" y="96" text-anchor="middle" fill="#181C22">LB</text>
          <path d="M240,92 L290,92" stroke="#565F6B" stroke-width="1.4" marker-end="url(#a5)"/>
          <text x="265" y="80" text-anchor="middle" font-size="9.5" fill="#565F6B">route</text>

          <rect x="290" y="70" width="90" height="44" rx="6" fill="#DDEFE4" stroke="#2F8F63"/>
          <text x="335" y="96" text-anchor="middle" fill="#1F6B47">Service A</text>
          <path d="M380,92 L430,92" stroke="#565F6B" stroke-width="1.4" marker-end="url(#a5)"/>
          <text x="405" y="80" text-anchor="middle" font-size="9" fill="#565F6B">cross-AZ</text>

          <rect x="430" y="70" width="90" height="44" rx="6" fill="#F6E9D3" stroke="#C8862B"/>
          <text x="475" y="96" text-anchor="middle" fill="#8C5E13">Service B</text>
          <path d="M520,92 L570,92" stroke="#565F6B" stroke-width="1.4" marker-end="url(#a5)"/>
          <text x="545" y="80" text-anchor="middle" font-size="9" fill="#565F6B">cross-region</text>

          <rect x="570" y="70" width="110" height="44" rx="6" fill="#F7E4E3" stroke="#C7454A" stroke-width="1.5"/>
          <text x="625" y="90" text-anchor="middle" fill="#8E2E32" font-size="11">Service C</text>
          <text x="625" y="105" text-anchor="middle" fill="#8E2E32" font-size="9.5">other region</text>
        </g>
        <text x="350" y="150" text-anchor="middle" font-family="Space Grotesk" font-size="12" fill="#565F6B">A single slow/misconfigured hop (e.g. a bad health check) can eject otherwise-healthy instances downstream.</text>
      </svg>
      <figcaption>Distributed tracing turns this row of boxes into a waterfall — the wide gaps are where the network, not the code, is spending time.</figcaption>
    </figure>

    <h4>Diagnosis</h4>
    <ul>
      <li>Compare "time in app code" vs. "time in transit" using distributed tracing — a trace waterfall showing large gaps between spans.</li>
      <li>Load balancer metrics: active connections, backend error rate, health check failures.</li>
      <li><code>ping</code> / <code>traceroute</code> / <code>mtr</code> for raw latency and packet loss where you can debug directly.</li>
      <li>Rule out DNS — slow or failing resolution often masquerades as a generic "connection timeout."</li>
    </ul>

    <h4>Fix / prevention</h4>
    <ul>
      <li>Reduce cross-service chattiness: batch requests, use gRPC/streaming instead of many round trips.</li>
      <li>Co-locate latency-sensitive services in the same AZ/region.</li>
      <li>Tune load balancer health check intervals/thresholds so transient blips don't cause mass instance ejection.</li>
      <li>Use connection keep-alive and HTTP/2 multiplexing to avoid repeated handshake overhead.</li>
      <li>Circuit breakers and timeouts on every outbound call, so one slow network path doesn't stall the whole request chain.</li>
    </ul>

    <h2 id="gc" class="mode-h2"><span class="modebadge">08</span>GC pauses</h2>
    <p>In managed-memory runtimes (JVM, .NET CLR, Go's GC), the garbage collector periodically pauses application threads ("stop-the-world") to reclaim memory. Under memory pressure or with a poorly tuned collector, these pauses grow long and frequent enough to show up as latency spikes — and in extreme cases, cause the process to be marked unhealthy by orchestration or a load balancer.</p>

    <h4>Why it happens</h4>
    <ul>
      <li>Heap sized too small for the actual working set, forcing frequent collections.</li>
      <li>High allocation rate — lots of short-lived object churn overwhelming the young generation.</li>
      <li>Memory leaks — objects that should be garbage but are still referenced, growing the old generation until a full GC is needed.</li>
      <li>An old/default collector (Parallel GC) used for a latency-sensitive service instead of a low-pause one (G1, ZGC, Shenandoah).</li>
      <li>Large heaps paradoxically making full-GC pauses worse if not tuned for concurrent/incremental collection.</li>
    </ul>

    <figure class="diagram">
      <svg viewBox="0 0 700 170" xmlns="http://www.w3.org/2000/svg" role="img" aria-labelledby="gc-title">
        <title id="gc-title">Request latency around a stop-the-world GC pause</title>
        <line x1="20" y1="130" x2="680" y2="130" stroke="#D8DBD1"/>
        <rect x="20" y="112" width="120" height="18" rx="3" fill="#DDEFE4" stroke="#2F8F63"/>
        <text x="80" y="105" text-anchor="middle" font-family="Space Grotesk" font-size="10.5" fill="#1F6B47">Request A (fast)</text>
        <rect x="150" y="112" width="120" height="18" rx="3" fill="#DDEFE4" stroke="#2F8F63"/>
        <text x="210" y="105" text-anchor="middle" font-family="Space Grotesk" font-size="10.5" fill="#1F6B47">Request B (fast)</text>

        <rect x="280" y="30" width="130" height="100" rx="3" fill="#F7E4E3" stroke="#C7454A" stroke-width="1.5"/>
        <text x="345" y="24" text-anchor="middle" font-family="Space Grotesk" font-size="11.5" font-weight="700" fill="#8E2E32">Stop-the-world GC pause</text>

        <rect x="280" y="60" width="180" height="18" rx="3" fill="#F6E9D3" stroke="#C8862B"/>
        <text x="370" y="53" text-anchor="middle" font-family="Space Grotesk" font-size="10.5" fill="#8C5E13">Request C — blocked, then spikes</text>
        <rect x="290" y="84" width="200" height="18" rx="3" fill="#F6E9D3" stroke="#C8862B"/>
        <text x="390" y="77" text-anchor="middle" font-family="Space Grotesk" font-size="10.5" fill="#8C5E13">Request D — blocked, then spikes</text>

        <text x="345" y="150" text-anchor="middle" font-family="IBM Plex Mono" font-size="11" fill="#565F6B">time →</text>
      </svg>
      <figcaption>The smoking gun in most postmortems: p99/p999 latency spikes line up exactly with GC pause timestamps.</figcaption>
    </figure>

    <h4>Diagnosis</h4>
    <ul>
      <li>GC logs (<code>-Xlog:gc*</code> on modern JVMs) showing pause duration and frequency.</li>
      <li>Correlate p99/p999 latency spikes with GC pause timestamps.</li>
      <li>Heap dumps / profilers (async-profiler, VisualVM) to find allocation hotspots or leaks.</li>
      <li>Kubernetes liveness-probe failures that line up exactly with long GC pauses — the process is "alive" but unresponsive during STW.</li>
    </ul>

    <h4>Fix / prevention</h4>
    <ul>
      <li>Choose a low-pause collector for latency-sensitive services — G1 as a solid default, ZGC/Shenandoah for very large heaps needing sub-millisecond pauses.</li>
      <li>Right-size the heap: too small causes GC thrashing, too large without a concurrent collector causes long full-GC pauses.</li>
      <li>Reduce allocation pressure — object pooling on very hot paths, avoiding unnecessary boxing, streaming instead of buffering large payloads.</li>
      <li>Fix leaks — common culprits are unbounded caches, listeners/callbacks never unregistered, ThreadLocals never cleared.</li>
      <li>Set liveness-probe thresholds with GC behavior in mind, so a normal (if long) pause doesn't trigger an unnecessary pod restart that compounds the problem.</li>
    </ul>

    <div class="pre-label">A reasonable G1 starting point</div>
    <pre><code>-Xms4g -Xmx4g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:+ParallelRefProcEnabled
-Xlog:gc*:file=/var/log/app/gc.log:time,level,tags</code></pre>

    <h2 id="checklist" class="mode-h2"><span class="modebadge">09</span>The interview diagnostic checklist</h2>
    <p>When an interviewer says "the service is slow, walk me through debugging it," this is the mental checklist worth narrating out loud, roughly in order:</p>

    <ol class="checklist">
      <li><span class="label">CPU check karo</span>Is CPU actually maxed out, or low despite high latency? Low CPU + high latency usually means <em>waiting</em> — locks, I/O, network — not computing.</li>
      <li><span class="label">Memory check karo</span>Is the process near its memory limit? Climbing steadily (a leak) or spiking in bursts (an allocation storm)?</li>
      <li><span class="label">Garbage collection pauses dekho</span>Correlate GC pause timestamps with latency spikes — often the fastest way to confirm or rule out GC as the culprit.</li>
      <li><span class="label">Network latency verify karo</span>Use distributed tracing to see how much time is in the network vs. application code. Check cross-AZ/region hops specifically.</li>
      <li><span class="label">Load balancer healthy hai ya nahi</span>Check LB-level metrics — are backend instances flapping between healthy and unhealthy? Are health check failures correlated with GC pauses or pool exhaustion, i.e. is the LB reacting to a downstream problem rather than causing one?</li>
    </ol>

    <div class="callout blue">
      <div class="callout-title"><span class="dot g"></span>What's actually being evaluated</div>
      Not whether you've memorized this list — whether you reason from symptom → correlated metrics → root cause → fix, and understand how these eight failure modes chain into and mask each other.
    </div>

    <h2 id="cascade" class="mode-h2"><span class="modebadge">10</span>How these failures cascade</h2>
    <p>In a real incident, these almost never happen in isolation. A typical chain:</p>

    <figure class="diagram">
      <svg viewBox="0 0 700 420" xmlns="http://www.w3.org/2000/svg" role="img" aria-labelledby="cascade-title">
        <title id="cascade-title">A traffic spike cascading through all eight failure modes and back to itself</title>
        <defs><marker id="a6" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" fill="#565F6B"/></marker></defs>
        <g font-family="Space Grotesk" font-size="11.5" text-anchor="middle">
          <rect x="270" y="10" width="160" height="38" rx="7" fill="#F6E9D3" stroke="#C8862B"/>
          <text x="350" y="34" fill="#8C5E13" font-weight="700">Traffic spike</text>

          <path d="M350,48 L350,78" stroke="#565F6B" stroke-width="1.3" marker-end="url(#a6)"/>
          <rect x="230" y="80" width="240" height="38" rx="7" fill="#F7E4E3" stroke="#C7454A"/>
          <text x="350" y="104" fill="#8E2E32">Cache miss storm / thundering herd</text>

          <path d="M350,118 L350,148" stroke="#565F6B" stroke-width="1.3" marker-end="url(#a6)"/>
          <rect x="230" y="150" width="240" height="38" rx="7" fill="#F7E4E3" stroke="#C7454A"/>
          <text x="350" y="174" fill="#8E2E32">Slow queries flood the DB</text>

          <path d="M350,188 L350,218" stroke="#565F6B" stroke-width="1.3" marker-end="url(#a6)"/>
          <rect x="230" y="220" width="240" height="38" rx="7" fill="#F7E4E3" stroke="#C7454A"/>
          <text x="350" y="244" fill="#8E2E32">Lock contention queues transactions</text>

          <path d="M350,258 L350,288" stroke="#565F6B" stroke-width="1.3" marker-end="url(#a6)"/>
          <rect x="230" y="290" width="240" height="38" rx="7" fill="#F7E4E3" stroke="#C7454A"/>
          <text x="350" y="314" fill="#8E2E32">Connection pool saturates fleet-wide</text>

          <path d="M350,328 L350,358" stroke="#565F6B" stroke-width="1.3" marker-end="url(#a6)"/>
          <rect x="230" y="360" width="240" height="38" rx="7" fill="#F7E4E3" stroke="#C7454A"/>
          <text x="350" y="384" fill="#8E2E32">GC worsens, LB ejects slow instances</text>

          <path d="M470,379 C 600,379 620,60 440,29" stroke="#C7454A" stroke-width="1.3" fill="none" marker-end="url(#a6)"/>
          <text x="600" y="200" font-family="Space Grotesk" font-size="11" fill="#8E2E32" transform="rotate(90 600 200)">remaining instances take more load →</text>
        </g>
      </svg>
      <figcaption>The loop closes on itself: ejecting slow instances concentrates load on the survivors, deepening the same spike that started it.</figcaption>
    </figure>

    <p>This is why a strong interview answer names the failure mode you'd check <em>first</em>, but also explains how you'd separate root cause from downstream symptom — for example: "pool exhaustion is often not a pool-sizing problem, it's a symptom of slow queries or lock contention upstream."</p>

    <h2 id="references"><span class="num">11</span>References</h2>
    <p>This guide draws on official documentation (AWS, Oracle/OpenJDK, PostgreSQL, MySQL) and independent engineering write-ups. For interview practice specifically, cross-reference against more than one prep source and real interview-experience threads — question patterns and interviewer expectations vary by company and change over time.</p>

    <div class="ref-group">
      <div class="group-title">Connection pool saturation</div>
      <ul>
        <li>HikariCP connection pool tuning guidance for Spring Boot</li>
        <li>Database connection pool tuning under traffic spikes (HikariCP + PostgreSQL)</li>
        <li>Production-oriented Hikari connection pool guides</li>
      </ul>
    </div>
    <div class="ref-group">
      <div class="group-title">Cache miss storm / thundering herd</div>
      <ul>
        <li>Thundering herd problem — general OS/networking background</li>
        <li>Cache stampede — background and terminology</li>
        <li>Cache stampede mitigation write-ups (Redis-focused)</li>
      </ul>
    </div>
    <div class="ref-group">
      <div class="group-title">Lock contention</div>
      <ul>
        <li>MySQL — <code>SHOW ENGINE INNODB STATUS</code> documentation</li>
        <li>PostgreSQL — <code>pg_locks</code> system view documentation</li>
      </ul>
    </div>
    <div class="ref-group">
      <div class="group-title">Hot partitions</div>
      <ul>
        <li>AWS DynamoDB Developer Guide — key-range throughput exceeded / hot partition mitigation</li>
        <li>DynamoDB hot partition explainers and partition/sharding write-ups</li>
      </ul>
    </div>
    <div class="ref-group">
      <div class="group-title">Slow queries</div>
      <ul>
        <li>PostgreSQL <code>EXPLAIN</code> documentation</li>
        <li>MySQL slow query log documentation</li>
      </ul>
    </div>
    <div class="ref-group">
      <div class="group-title">Network bottlenecks</div>
      <ul>
        <li>AWS Well-Architected Framework — Performance Efficiency pillar</li>
      </ul>
    </div>
    <div class="ref-group">
      <div class="group-title">GC pauses</div>
      <ul>
        <li>Oracle JVM garbage collection tuning guide</li>
        <li>G1 garbage collector overview (Oracle)</li>
        <li>ZGC — OpenJDK wiki</li>
      </ul>
    </div>

  </article>
</div>

<hr class="rule" style="max-width:1180px; margin:0 auto;">

<footer>
  <p style="font-family:'Space Grotesk',sans-serif; font-size:13px; text-transform:uppercase; letter-spacing:.1em; color:var(--ink-soft);">A note on sourcing</p>
  <p>This guide pulls from official documentation (AWS, Oracle/OpenJDK, PostgreSQL, MySQL) and independent engineering blogs. For system design interview practice specifically, cross-reference against multiple prep resources and real interview-experience threads — expectations vary by company and change over time.</p>
</footer>

<script>
  const links = Array.from(document.querySelectorAll('.toc a'));
  const sections = links.map(a => document.querySelector(a.getAttribute('href')));
  function onScroll(){
    let idx = 0;
    const y = window.scrollY + 100;
    sections.forEach((s, i) => { if(s && s.offsetTop <= y) idx = i; });
    links.forEach((a,i) => a.classList.toggle('active', i === idx));
  }
  document.addEventListener('scroll', onScroll, {passive:true});
  onScroll();
</script>
</body>
</html>
