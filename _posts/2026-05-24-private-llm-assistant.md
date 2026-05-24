<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Building a Private LLM Security Assistant | botesjuan</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;700&family=Syne:wght@400;600;700;800&display=swap');

  :root {
    --bg:        #0a0c0f;
    --bg2:       #0f1218;
    --bg3:       #141920;
    --border:    #1e2730;
    --green:     #00ff88;
    --green-dim: #00cc6a;
    --cyan:      #00d4ff;
    --amber:     #ffaa00;
    --red:       #ff4455;
    --purple:    #b388ff;
    --text:      #c8d6e5;
    --text-dim:  #6b7c93;
    --text-faint:#3a4a5a;
  }

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  html { scroll-behavior: smooth; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'JetBrains Mono', monospace;
    font-size: 14px;
    line-height: 1.8;
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* Scanline overlay */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background: repeating-linear-gradient(
      0deg,
      transparent,
      transparent 2px,
      rgba(0,0,0,0.03) 2px,
      rgba(0,0,0,0.03) 4px
    );
    pointer-events: none;
    z-index: 9999;
  }

  /* Grid background */
  body::after {
    content: '';
    position: fixed;
    inset: 0;
    background-image:
      linear-gradient(rgba(0,255,136,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,255,136,0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 0;
  }

  .wrap {
    position: relative;
    z-index: 1;
    max-width: 860px;
    margin: 0 auto;
    padding: 0 24px 80px;
  }

  /* ── HEADER ── */
  header {
    padding: 48px 0 40px;
    border-bottom: 1px solid var(--border);
    margin-bottom: 48px;
  }

  .site-tag {
    font-size: 11px;
    color: var(--green);
    letter-spacing: 0.2em;
    text-transform: uppercase;
    margin-bottom: 20px;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .site-tag::before {
    content: '';
    display: inline-block;
    width: 8px; height: 8px;
    background: var(--green);
    border-radius: 50%;
    animation: pulse 2s ease-in-out infinite;
  }

  @keyframes pulse {
    0%,100% { opacity:1; box-shadow: 0 0 0 0 rgba(0,255,136,0.4); }
    50%      { opacity:0.6; box-shadow: 0 0 0 6px rgba(0,255,136,0); }
  }

  h1 {
    font-family: 'Syne', sans-serif;
    font-size: clamp(28px, 5vw, 48px);
    font-weight: 800;
    line-height: 1.1;
    color: #fff;
    margin-bottom: 16px;
  }

  h1 span { color: var(--green); }

  .meta {
    display: flex;
    flex-wrap: wrap;
    gap: 16px;
    font-size: 12px;
    color: var(--text-dim);
    margin-top: 20px;
  }

  .meta span {
    display: flex;
    align-items: center;
    gap: 6px;
  }

  .meta span::before {
    content: '▸';
    color: var(--green);
  }

  /* ── SECTION HEADINGS ── */
  h2 {
    font-family: 'Syne', sans-serif;
    font-size: 22px;
    font-weight: 700;
    color: #fff;
    margin: 48px 0 20px;
    padding-left: 16px;
    border-left: 3px solid var(--green);
    display: flex;
    align-items: center;
    gap: 10px;
  }

  h2 .num {
    font-family: 'JetBrains Mono', monospace;
    font-size: 12px;
    color: var(--green);
    font-weight: 400;
    opacity: 0.7;
  }

  h3 {
    font-family: 'Syne', sans-serif;
    font-size: 15px;
    font-weight: 600;
    color: var(--cyan);
    margin: 28px 0 10px;
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }

  p {
    color: var(--text);
    margin-bottom: 14px;
  }

  /* ── ARCHITECTURE DIAGRAM ── */
  .arch-diagram {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 32px;
    margin: 32px 0;
    position: relative;
    overflow: hidden;
  }

  .arch-diagram::before {
    content: 'ARCHITECTURE';
    position: absolute;
    top: 12px; right: 16px;
    font-size: 10px;
    color: var(--text-faint);
    letter-spacing: 0.2em;
  }

  .arch-title {
    font-size: 11px;
    color: var(--green);
    letter-spacing: 0.15em;
    text-transform: uppercase;
    margin-bottom: 24px;
  }

  /* Node boxes */
  .node {
    background: var(--bg3);
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 16px 20px;
    margin: 12px 0;
    position: relative;
    transition: border-color 0.2s;
  }

  .node:hover { border-color: var(--green-dim); }

  .node-label {
    font-size: 10px;
    letter-spacing: 0.15em;
    text-transform: uppercase;
    margin-bottom: 6px;
  }

  .node-name {
    font-family: 'Syne', sans-serif;
    font-size: 16px;
    font-weight: 700;
    color: #fff;
    margin-bottom: 4px;
  }

  .node-detail {
    font-size: 11px;
    color: var(--text-dim);
  }

  .node-ip {
    position: absolute;
    top: 12px; right: 14px;
    font-size: 11px;
    color: var(--amber);
    font-family: 'JetBrains Mono', monospace;
  }

  .node-badge {
    display: inline-block;
    font-size: 10px;
    padding: 2px 8px;
    border-radius: 3px;
    margin-top: 8px;
    margin-right: 4px;
    font-weight: 500;
  }

  .badge-green  { background: rgba(0,255,136,0.1); color: var(--green); border: 1px solid rgba(0,255,136,0.2); }
  .badge-cyan   { background: rgba(0,212,255,0.1); color: var(--cyan);  border: 1px solid rgba(0,212,255,0.2); }
  .badge-amber  { background: rgba(255,170,0,0.1); color: var(--amber); border: 1px solid rgba(255,170,0,0.2); }
  .badge-purple { background: rgba(179,136,255,0.1);color:var(--purple);border: 1px solid rgba(179,136,255,0.2);}
  .badge-red    { background: rgba(255,68,85,0.1);  color: var(--red);  border: 1px solid rgba(255,68,85,0.2); }

  .connector {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 4px 20px;
    font-size: 11px;
    color: var(--text-faint);
  }

  .connector::before, .connector::after {
    content: '';
    flex: 1;
    height: 1px;
    background: var(--border);
  }

  /* ── GOAL / STATUS CARDS ── */
  .goals {
    display: grid;
    gap: 10px;
    margin: 24px 0;
  }

  .goal {
    display: grid;
    grid-template-columns: 40px 1fr auto;
    align-items: center;
    gap: 14px;
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 14px 18px;
    transition: border-color 0.2s;
  }

  .goal:hover { border-color: var(--border); }

  .goal-num {
    font-size: 11px;
    color: var(--text-faint);
    text-align: center;
  }

  .goal-text {
    font-size: 13px;
    color: var(--text);
  }

  .goal-text strong {
    color: #fff;
    display: block;
    font-size: 13px;
    margin-bottom: 2px;
  }

  .goal-text span {
    font-size: 11px;
    color: var(--text-dim);
  }

  .status {
    font-size: 10px;
    padding: 3px 10px;
    border-radius: 3px;
    white-space: nowrap;
    font-weight: 500;
    letter-spacing: 0.05em;
  }

  .done     { background: rgba(0,255,136,0.12); color: var(--green); border: 1px solid rgba(0,255,136,0.25); }
  .active   { background: rgba(0,212,255,0.12); color: var(--cyan);  border: 1px solid rgba(0,212,255,0.25); }
  .planned  { background: rgba(107,124,147,0.12); color: var(--text-dim); border: 1px solid var(--border); }
  .future   { background: rgba(179,136,255,0.12); color: var(--purple); border: 1px solid rgba(179,136,255,0.25); }

  /* ── CODE BLOCKS ── */
  .code-block {
    background: #060809;
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 20px;
    margin: 16px 0;
    overflow-x: auto;
    position: relative;
  }

  .code-block::before {
    content: attr(data-lang);
    position: absolute;
    top: 8px; right: 12px;
    font-size: 10px;
    color: var(--text-faint);
    letter-spacing: 0.1em;
    text-transform: uppercase;
  }

  .code-block code {
    font-family: 'JetBrains Mono', monospace;
    font-size: 12.5px;
    line-height: 1.7;
    color: #8bb8d4;
    white-space: pre;
    display: block;
  }

  .c-green  { color: var(--green); }
  .c-amber  { color: var(--amber); }
  .c-cyan   { color: var(--cyan); }
  .c-purple { color: var(--purple); }
  .c-dim    { color: var(--text-faint); }
  .c-red    { color: var(--red); }

  /* ── HARDWARE SPECS ── */
  .spec-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 12px;
    margin: 20px 0;
  }

  @media (max-width: 600px) { .spec-grid { grid-template-columns: 1fr; } }

  .spec-card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 16px;
  }

  .spec-card-label {
    font-size: 10px;
    color: var(--text-faint);
    letter-spacing: 0.15em;
    text-transform: uppercase;
    margin-bottom: 8px;
  }

  .spec-card-value {
    font-size: 14px;
    color: #fff;
    font-weight: 500;
  }

  .spec-card-sub {
    font-size: 11px;
    color: var(--text-dim);
    margin-top: 3px;
  }

  /* ── CALLOUT ── */
  .callout {
    border-left: 3px solid var(--amber);
    background: rgba(255,170,0,0.05);
    border-radius: 0 6px 6px 0;
    padding: 16px 20px;
    margin: 20px 0;
    font-size: 13px;
    color: var(--text);
  }

  .callout.improvement {
    border-color: var(--cyan);
    background: rgba(0,212,255,0.05);
  }

  .callout-label {
    font-size: 10px;
    letter-spacing: 0.15em;
    text-transform: uppercase;
    margin-bottom: 8px;
    font-weight: 600;
  }

  .callout .callout-label { color: var(--amber); }
  .callout.improvement .callout-label { color: var(--cyan); }

  /* ── LEARNING SECTION ── */
  .learning-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 12px;
    margin: 20px 0;
  }

  .learning-card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 16px;
    text-align: center;
  }

  .learning-card .icon {
    font-size: 24px;
    margin-bottom: 10px;
    display: block;
  }

  .learning-card .lc-title {
    font-family: 'Syne', sans-serif;
    font-size: 13px;
    font-weight: 600;
    color: #fff;
    margin-bottom: 6px;
  }

  .learning-card .lc-desc {
    font-size: 11px;
    color: var(--text-dim);
    line-height: 1.6;
  }

  /* ── FOOTER ── */
  footer {
    border-top: 1px solid var(--border);
    padding-top: 32px;
    margin-top: 64px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 12px;
    font-size: 11px;
    color: var(--text-faint);
  }

  footer a {
    color: var(--green);
    text-decoration: none;
  }

  footer a:hover { text-decoration: underline; }

  /* ── DIVIDER ── */
  .divider {
    border: none;
    border-top: 1px solid var(--border);
    margin: 40px 0;
  }

  /* ── INLINE CODE ── */
  code {
    background: rgba(0,255,136,0.08);
    border: 1px solid rgba(0,255,136,0.15);
    border-radius: 3px;
    padding: 1px 6px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 12px;
    color: var(--green);
  }

  a { color: var(--cyan); text-decoration: none; }
  a:hover { text-decoration: underline; }

  ul, ol {
    padding-left: 20px;
    margin-bottom: 14px;
  }

  li {
    color: var(--text);
    margin-bottom: 6px;
    font-size: 13px;
  }

  li::marker { color: var(--green); }

  /* Fade-in animation */
  @keyframes fadeUp {
    from { opacity:0; transform: translateY(16px); }
    to   { opacity:1; transform: translateY(0); }
  }

  .wrap > * {
    animation: fadeUp 0.5s ease both;
  }

</style>
</head>
<body>
<div class="wrap">

  <!-- HEADER -->
  <header>
    <div class="site-tag">botesjuan.github.io &nbsp;/&nbsp; infosec projects</div>
    <h1>Building a <span>Private LLM</span><br>Security Assistant</h1>
    <p style="color:var(--text-dim); font-size:13px; margin-top:14px; max-width:620px;">
      A fully private, GPU-accelerated AI security assistant running on dedicated local hardware —
      no cloud dependency, no data leakage, full tool execution capability for professional penetration testing engagements.
    </p>
    <div class="meta">
      <span>Juan Botes — Senior Penetration Tester, Integrity360</span>
      <span>Cape Town, South Africa</span>
      <span>May 2026</span>
      <span>Status: Active Build</span>
    </div>
  </header>

  <!-- SECTION 1: THE GOAL -->
  <h2><span class="num">01 //</span> Project Goal</h2>
  <p>
    The goal is to build a fully private, self-hosted AI security assistant that operates like
    <strong style="color:#fff">Claude Code</strong> — an agentic CLI tool that reasons over goals, executes shell commands,
    edits files, and chains tool output into further decisions — but running entirely on local hardware
    with a private LLM as the backend. Zero API costs. Zero data leaving the network.
  </p>
  <p>
    This project has three primary deliverables:
  </p>
  <ul>
    <li><strong style="color:#fff">Private CLI Agent</strong> — A terminal client on the Ubuntu desktop that connects to the private LLM backend and can execute commands, edit files, and run pentest tools autonomously based on natural language prompts.</li>
    <li><strong style="color:#fff">Enhanced Web Frontend</strong> — Upgrade the existing <code>llm_prompt.php</code> web app on groupservice.co.za to support file upload, image upload, image-to-text, and (future) text-to-image using the private LLM backend.</li>
    <li><strong style="color:#fff">Private LLM Backend</strong> — A dedicated headless Ubuntu Server node with GPU-accelerated inference, persistent memory, and a sandboxed tool execution environment for security tooling.</li>
  </ul>

  <!-- SECTION 2: ARCHITECTURE -->
  <h2><span class="num">02 //</span> System Architecture</h2>

  <div class="arch-diagram">
    <div class="arch-title">▸ network topology</div>

    <!-- Internet layer -->
    <div class="node" style="border-color:rgba(0,255,136,0.3);">
      <div class="node-label" style="color:var(--green)">Public Internet</div>
      <div class="node-name">groupservice.co.za</div>
      <div class="node-detail">HTTPS · Let's Encrypt SSL · MFA-enabled web portal</div>
      <div style="margin-top:10px">
        <span class="node-badge badge-green">llm_prompt.php</span>
        <span class="node-badge badge-green">llm_audit.log</span>
        <span class="node-badge badge-amber">MFA</span>
      </div>
    </div>

    <div class="connector">↕ HTTPS / SSL · Apache reverse proxy</div>

    <!-- Pi layer -->
    <div class="node">
      <div class="node-ip"><front-end-ip></div>
      <div class="node-label" style="color:var(--cyan)">Public Frontend Node</div>
      <div class="node-name">Raspberry Pi 4</div>
      <div class="node-detail">Apache · SSL termination · Postfix · SIEM dashboard · Audit logging</div>
      <div style="margin-top:10px">
        <span class="node-badge badge-cyan">Apache</span>
        <span class="node-badge badge-cyan">SSL</span>
        <span class="node-badge badge-cyan">SIEM</span>
        <span class="node-badge badge-amber">llm_admin / MFA</span>
      </div>
    </div>

    <div class="connector">↕ Internal LAN · Ports 80 / 443 only · Firewall restricted</div>

    <!-- LLM Backend -->
    <div class="node" style="border-color:rgba(179,136,255,0.3);">
      <div class="node-ip"><llm-box-ip></div>
      <div class="node-label" style="color:var(--purple)">Private LLM Backend — z490</div>
      <div class="node-name">Ubuntu Server 24.04 LTS</div>
      <div class="node-detail">MSI Z490-A PRO · i7-10700 · 64GB DDR4 · RTX 4060 Ti 16GB VRAM · Headless</div>
      <div style="margin-top:10px">
        <span class="node-badge badge-purple">Ollama :11434</span>
        <span class="node-badge badge-purple">Open WebUI :3000</span>
        <span class="node-badge badge-purple">ChromaDB</span>
        <span class="node-badge badge-red">LAN only</span>
      </div>
    </div>

    <div class="connector">↕ LAN · SSH · API calls</div>

    <!-- Desktop CLI -->
    <div class="node">
      <div class="node-ip"><cli-client-ip></div>
      <div class="node-label" style="color:var(--amber)">Internal CLI Client</div>
      <div class="node-name">Ubuntu Desktop</div>
      <div class="node-detail">Private Claude Code CLI agent · Kali VM · Direct LLM API access for testing</div>
      <div style="margin-top:10px">
        <span class="node-badge badge-amber">CLI Agent</span>
        <span class="node-badge badge-amber">Kali VM</span>
        <span class="node-badge badge-green">ssh user@<llm-box-ip></span>
      </div>
    </div>

  </div>

  <!-- SECTION 3: HARDWARE -->
  <h2><span class="num">03 //</span> Hardware</h2>

  <h3>Private LLM Backend — z490</h3>
  <div class="spec-grid">
    <div class="spec-card">
      <div class="spec-card-label">Motherboard</div>
      <div class="spec-card-value">MSI Z490-A PRO</div>
      <div class="spec-card-sub">MS-7C75 · Comet Lake S</div>
    </div>
    <div class="spec-card">
      <div class="spec-card-label">CPU</div>
      <div class="spec-card-value">Intel i7-10700</div>
      <div class="spec-card-sub">8c/16t · 2.9GHz base · 4.8GHz boost · AVX2</div>
    </div>
    <div class="spec-card">
      <div class="spec-card-label">System Memory</div>
      <div class="spec-card-value">64GB DDR4</div>
      <div class="spec-card-sub">4 × 16GB · 2133MHz · Quad channel</div>
    </div>
    <div class="spec-card">
      <div class="spec-card-label">GPU — Primary Compute</div>
      <div class="spec-card-value">NVIDIA RTX 4060 Ti</div>
      <div class="spec-card-sub">16GB GDDR6 VRAM · CUDA · Ada Lovelace · No display</div>
    </div>
    <div class="spec-card">
      <div class="spec-card-label">Storage — OS</div>
      <div class="spec-card-value">Samsung 960 EVO 250GB</div>
      <div class="spec-card-sub">NVMe PCIe · Ubuntu Server 24.04</div>
    </div>
    <div class="spec-card">
      <div class="spec-card-label">Network</div>
      <div class="spec-card-value">RTL8125 2.5GbE</div>
      <div class="spec-card-sub">Static IP · <llm-box-ip> · LAN only</div>
    </div>
    <div class="spec-card">
      <div class="spec-card-label">Display (iGFX)</div>
      <div class="spec-card-value">Intel UHD 630</div>
      <div class="spec-card-sub">Set as primary in BIOS · Headless operation</div>
    </div>
    <div class="spec-card">
      <div class="spec-card-label">Access</div>
      <div class="spec-card-value">SSH key only</div>
      <div class="spec-card-sub">user@<llm-box-ip> · ed25519 · Password auth disabled</div>
    </div>
  </div>

  <div class="callout">
    <div class="callout-label">⚠ Storage Note</div>
    The 250GB NVMe is tight for multiple large LLM models. A 14B model at Q4 quantization uses ~8–9GB.
    A second NVMe (1–2TB) is recommended for <code>/models</code> and <code>/data</code> mount points
    to avoid OS partition pressure as the model library grows.
  </div>

  <!-- SECTION 4: STACK -->
  <h2><span class="num">04 //</span> Software Stack</h2>

  <h3>LLM Backend Services</h3>
  <div class="code-block" data-lang="stack">
<code><span class="c-green">LLM Runtime</span>
  Ollama                    <span class="c-dim"># GPU-accelerated model serving</span>
  OLLAMA_BASE_URL           <span class="c-amber">http://<llm-box-ip>:11434</span>

<span class="c-green">Active Models</span>
  hf.co/TrevorJS/gemma-4-E2B-it-uncensored-GGUF:Q4_K_M   <span class="c-dim"># primary</span>
  qwen2.5:14b                                              <span class="c-dim"># tool-call capable</span>

<span class="c-green">Web UI</span>
  Open WebUI                <span class="c-dim"># Docker · port 3000 · LAN only</span>

<span class="c-green">Vector Memory</span>
  ChromaDB                  <span class="c-dim"># persistent RAG store</span>

<span class="c-green">Agent Framework</span>
  Python ReAct loop         <span class="c-dim"># reason → act → observe → repeat</span>

<span class="c-green">Tool Sandbox</span>
  Docker containers         <span class="c-dim"># isolated tool execution</span>

<span class="c-green">Firewall</span>
  ufw                       <span class="c-dim"># ports 22/80/443 from <front-end-ip> only</span></code>
  </div>

  <h3>Public Frontend — groupservice.co.za</h3>
  <div class="code-block" data-lang="stack">
<code><span class="c-green">Host</span>         Raspberry Pi 4 · <front-end-ip>
<span class="c-green">Web Server</span>   Apache · Let's Encrypt SSL
<span class="c-green">Auth</span>         llm_admin · MFA enabled
<span class="c-green">App</span>          /llm_prompt.php  →  proxies to Ollama API on z490
<span class="c-green">Audit Log</span>    /var/www/groupservice/logs/llm_audit.log
<span class="c-green">Email</span>        Postfix (groupservice.co.za MX)
<span class="c-green">SIEM</span>         Custom dashboard (self-hosted)</code>
  </div>

  <!-- SECTION 5: BUILD GOALS -->
  <h2><span class="num">05 //</span> Build Goals &amp; Status</h2>

  <div class="goals">

    <div class="goal">
      <div class="goal-num">01</div>
      <div class="goal-text">
        <strong>Hardware migration</strong>
        <span>Removed iomemory cards · Installed RTX 4060 Ti · Swapped GT 960 to desktop</span>
      </div>
      <span class="status done">✓ DONE</span>
    </div>

    <div class="goal">
      <div class="goal-num">02</div>
      <div class="goal-text">
        <strong>Ubuntu Server 24.04 LTS install</strong>
        <span>Wiped Proxmox LVM · Clean headless install · SSH key auth · Static IP</span>
      </div>
      <span class="status done">✓ DONE</span>
    </div>

    <div class="goal">
      <div class="goal-num">03</div>
      <div class="goal-text">
        <strong>NVIDIA driver + CUDA + Ollama</strong>
        <span>RTX 4060 Ti compute-only · Models loaded · API live on :11434</span>
      </div>
      <span class="status done">✓ DONE</span>
    </div>

    <div class="goal">
      <div class="goal-num">04</div>
      <div class="goal-text">
        <strong>Open WebUI + Docker</strong>
        <span>Web interface on :3000 · LAN restricted · Docker sandboxing enabled</span>
      </div>
      <span class="status done">✓ DONE</span>
    </div>

    <div class="goal">
      <div class="goal-num">05</div>
      <div class="goal-text">
        <strong>Pentest tool suite</strong>
        <span>nmap · nuclei · ffuf · amass · subfinder · httpx · Docker-sandboxed</span>
      </div>
      <span class="status done">✓ DONE</span>
    </div>

    <div class="goal">
      <div class="goal-num">06</div>
      <div class="goal-text">
        <strong>Python ReAct agent framework + tool manifest</strong>
        <span>LLM reasons → calls tool → gets output → decides next step → loops</span>
      </div>
      <span class="status active">⬡ IN PROGRESS</span>
    </div>

    <div class="goal">
      <div class="goal-num">07</div>
      <div class="goal-text">
        <strong>Private CLI Agent — "Private Claude Code"</strong>
        <span>Terminal client on Ubuntu desktop · Connects to z490 LLM API · Full file + shell access</span>
      </div>
      <span class="status active">⬡ IN PROGRESS</span>
    </div>

    <div class="goal">
      <div class="goal-num">08</div>
      <div class="goal-text">
        <strong>ChromaDB vector store</strong>
        <span>Persistent memory · RAG over pentest notes, CVEs, client reports</span>
      </div>
      <span class="status planned">◻ PLANNED</span>
    </div>

    <div class="goal">
      <div class="goal-num">09</div>
      <div class="goal-text">
        <strong>Web frontend upgrade — file + image upload</strong>
        <span>llm_prompt.php enhanced · File upload · Image upload · Image-to-text via LLM backend</span>
      </div>
      <span class="status planned">◻ PLANNED</span>
    </div>

    <div class="goal">
      <div class="goal-num">10</div>
      <div class="goal-text">
        <strong>Text-to-image generation</strong>
        <span>Stable Diffusion / ComfyUI on RTX 4060 Ti · Integrated into web frontend</span>
      </div>
      <span class="status future">◈ FUTURE</span>
    </div>

    <div class="goal">
      <div class="goal-num">11</div>
      <div class="goal-text">
        <strong>Second NVMe storage expansion</strong>
        <span>Dedicated /models and /data mount · Expand model library beyond 14B</span>
      </div>
      <span class="status future">◈ FUTURE</span>
    </div>

    <div class="goal">
      <div class="goal-num">12</div>
      <div class="goal-text">
        <strong>vLLM migration + OpenAI-compatible API layer</strong>
        <span>Better batching · Full tool-call spec · Multi-client support</span>
      </div>
      <span class="status future">◈ FUTURE</span>
    </div>

  </div>

  <!-- SECTION 6: PRIVATE CLI AGENT -->
  <h2><span class="num">06 //</span> Private CLI Agent Design</h2>

  <p>The private CLI agent is a Python tool running on the Ubuntu desktop (<code><cli-client-ip</code>) that mirrors the Claude Code experience — but uses the local LLM box as its brain.</p>

  <div class="code-block" data-lang="flow">
<code><span class="c-green">You</span> (terminal prompt)
  │
  ▼
<span class="c-cyan">agent.py</span>  ←  reads AGENT.md (permissions + context)
  │
  ├─ sends goal to <span class="c-amber">http://<llm-box-ip>:11434/api/chat</span>
  │
  ▼
<span class="c-purple">LLM (gemma-4 / qwen2.5:14b)</span>
  │   reasons about goal
  │   decides which tool to call
  │   returns structured tool_call JSON
  │
  ▼
<span class="c-cyan">Tool Executor</span>
  ├── run_shell(cmd)         <span class="c-dim"># executes on desktop or via SSH to z490</span>
  ├── read_file(path)        <span class="c-dim"># reads any file on desktop</span>
  ├── write_file(path, data) <span class="c-dim"># creates / edits files</span>
  ├── run_nmap(target)       <span class="c-dim"># pentest tools</span>
  ├── run_nuclei(url)
  ├── run_ffuf(url, wl)
  └── run_subfinder(domain)
  │
  ▼
<span class="c-green">Output fed back to LLM</span>
  │   reasons over result
  │   decides next action
  │
  └─ loop until goal achieved → <span class="c-amber">final report to terminal</span></code>
  </div>

  <div class="callout improvement">
    <div class="callout-label">💡 Improvement over current setup</div>
    Rather than plain Ollama with no tool schema, use the <strong>OpenAI-compatible tool-call format</strong>
    that Ollama supports natively via <code>/api/chat</code> with a <code>tools</code> array.
    Models like <code>qwen2.5:14b</code> honour this spec reliably, giving structured JSON tool calls
    instead of regex-parsed text — much more robust for agentic loops.
  </div>

  <!-- SECTION 7: IMPROVEMENTS -->
  <h2><span class="num">07 //</span> Suggested Improvements</h2>

  <h3>Model Selection</h3>
  <p>The current model <code>gemma-4-E2B-it-uncensored-GGUF:Q4_K_M</code> is good for unrestricted output but was not fine-tuned for structured tool-call JSON. Consider a two-model approach:</p>
  <ul>
    <li><strong style="color:#fff">qwen2.5:14b</strong> — primary agentic model, native tool-call support, fits in 16GB VRAM, strong at structured output and multi-step reasoning.</li>
    <li><strong style="color:#fff">gemma-4-uncensored</strong> — secondary model for creative, unrestricted content generation and red team scenario planning where the primary model may be too cautious.</li>
  </ul>

  <h3>Storage Architecture</h3>
  <p>Add a second NVMe and mount as <code>/models</code> and <code>/data</code> to separate model storage from the OS. This allows the OS drive to be wiped/rebuilt without losing downloaded models and ChromaDB state.</p>

  <h3>API Security</h3>
  <p>The Ollama API on <code>:11434</code> currently has no authentication — protected only by the firewall. Add an API key header via a local nginx reverse proxy on z490 itself, so even LAN requests require a key. This follows defence-in-depth and matters when the CLI client is used from the Ubuntu desktop at <code>.51</code>.</p>

  <h3>Audit Logging</h3>
  <p>Extend <code>/var/www/groupservice/logs/llm_audit.log</code> to include: model used, token count, tool calls executed, and source IP. This is essential for professional accountability during client engagements and aligns with POPIA data handling obligations.</p>

  <h3>POPIA / Data Sanitisation</h3>
  <p>Client PII must never transit through the LLM — even a local one if logs are kept. Port the existing <code>sanitise.py</code> and <code>claude_guard.sh</code> tooling from the Kali VM to the CLI agent pipeline, running sanitisation before any prompt is sent to the backend.</p>

  <!-- SECTION 8: ANTHROPIC ACADEMY -->
  <h2><span class="num">08 //</span> Learning Integration — Anthropic Academy</h2>

  <p>This project is being built in parallel with structured learning from <a href="https://academy.anthropic.com" target="_blank">Anthropic Academy</a>. The following modules directly inform the agent architecture decisions:</p>

  <div class="learning-grid">
    <div class="learning-card">
      <span class="icon">🧠</span>
      <div class="lc-title">Prompt Engineering</div>
      <div class="lc-desc">Structuring tool-call prompts, system instructions, and ReAct-style reasoning chains for reliable agentic behaviour.</div>
    </div>
    <div class="learning-card">
      <span class="icon">🔧</span>
      <div class="lc-title">Tool Use & Function Calling</div>
      <div class="lc-desc">Designing tool manifests, parsing structured JSON responses, and building robust agent loops with error recovery.</div>
    </div>
    <div class="learning-card">
      <span class="icon">📚</span>
      <div class="lc-title">RAG & Memory</div>
      <div class="lc-desc">Building ChromaDB-backed retrieval pipelines over pentest notes, CVE feeds, and past engagement reports.</div>
    </div>
    <div class="learning-card">
      <span class="icon">🔒</span>
      <div class="lc-title">AI Safety in Context</div>
      <div class="lc-desc">Understanding guardrails, prompt injection risks in agentic systems, and responsible AI use in offensive security.</div>
    </div>
  </div>

  <div class="callout">
    <div class="callout-label">📌 Key Insight from Academy</div>
    The most important lesson applicable to this build: <strong style="color:#fff">an agent is only as reliable as its tool schema.</strong>
    Vague tool descriptions cause models to hallucinate calls. Every tool in the manifest must have a precise
    description, typed parameters, and example outputs — the model reads these at inference time to decide
    what to call and how to call it.
  </div>

  <!-- SECTION 9: NEXT STEPS -->
  <h2><span class="num">09 //</span> Next Steps</h2>

  <ol>
    <li>Complete Python ReAct agent loop with OpenAI-compatible tool-call schema against Ollama API.</li>
    <li>Build the CLI entrypoint (<code>agent.py</code>) with <code>AGENT.md</code> context loading — mirrors the Claude Code CLAUDE.md pattern.</li>
    <li>Test full agentic loop: <em>"enumerate subdomains for target.co.za, find open ports, run nuclei, summarise findings"</em> — zero manual intervention.</li>
    <li>Add second NVMe storage for model library expansion.</li>
    <li>Upgrade <code>llm_prompt.php</code> on the Pi with file upload and image-to-text support via multimodal model endpoint.</li>
    <li>Integrate POPIA data sanitisation layer into agent pipeline before production use on client engagements.</li>
    <li>Document each completed phase as a follow-up post on this blog.</li>
  </ol>

  <hr class="divider">

  <!-- FOOTER -->
  <footer>
    <div>
      <a href="https://botesjuan.github.io" target="_blank">botesjuan.github.io</a>
      &nbsp;·&nbsp;
      <a href="https://www.groupservice.co.za/llm_prompt.php" target="_blank">groupservice.co.za/llm_prompt.php</a>
    </div>
    <div>Juan Botes · OSCP · CISSP · BSCP · CPTS · CEH · <span style="color:var(--cyan)">CRTO (pursuing)</span></div>
  </footer>

</div>
</body>
</html>
