<p align="center">
  <img src="nexus1.png" alt="NEXUS Dashboard" width="100%">
</p>

<h1 align="center">NEXUS - AI Agent Firewall</h1>

<p align="center">
  <strong>Open-source security layer for AI agents. Intercepts every tool call. Detects threats in real-time. Zero code changes to your agent.</strong>
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> &bull;
  <a href="#how-it-works">How It Works</a> &bull;
  <a href="#detection-engines">Detection Engines</a> &bull;
  <a href="#dashboard">Dashboard</a> &bull;
  <a href="#compliance">Compliance</a> &bull;
  <a href="#api-reference">API Reference</a> &bull;
  <a href="#live-demo">Live Demo</a>
</p>

---

## The Problem

AI agents are powerful -- they can browse the web, read files, execute code, and send emails. But what happens when a malicious website injects hidden instructions into an agent's context? The agent becomes a weapon:

1. **Prompt injection** -- A webpage tells the agent to steal secrets
2. **Data theft** -- The agent reads `.env` files, API keys, credentials
3. **Exfiltration** -- The agent emails everything to `attacker@evil.com`

This entire attack chain happens in seconds, silently, with no visibility.

**NEXUS sits between your AI agent and its tools**, inspecting every call in real-time. It detects prompt injection, blocks access to sensitive files, flags secret exposure, and catches data exfiltration -- all before damage is done.

---

## Architecture

<p align="center">
  <img src="architecture.png" alt="NEXUS Architecture" width="90%">
</p>

NEXUS operates as an **MCP (Model Context Protocol) proxy**. Any MCP-compatible agent (Claude, LangChain, CrewAI, custom agents) connects to NEXUS instead of directly to tools. NEXUS intercepts, analyzes, and routes every tool call.

```
AI Agent  ──>  NEXUS Firewall  ──>  MCP Tool Servers
                    │
                    ├── Threat Detection (injection, DLP, exfiltration)
                    ├── Policy Engine (allow, block, monitor)
                    ├── Audit Trail (hash-chained, tamper-proof)
                    └── Real-Time Dashboard + Alerts
```

**Drop-in deployment** -- no code changes to the agent. Point your agent at NEXUS, and NEXUS handles the rest.

---

## How It Works

Every tool call goes through a 6-stage pipeline:

| Stage | What Happens |
|-------|-------------|
| **1. Intercept** | Agent sends MCP request to NEXUS proxy |
| **2. Pre-Analysis** | Threat detection scans request for sensitive file access, suspicious URLs, privilege escalation |
| **3. Policy Check** | Policy engine evaluates rules -- ALLOW, BLOCK, or MONITOR |
| **4. Route** | If allowed, NEXUS forwards request to the actual tool server |
| **5. Post-Analysis** | Response scanned for prompt injection, PII, secrets, exfiltration patterns |
| **6. Audit** | Event recorded to hash-chained audit trail, alerts generated, dashboard updated via WebSocket |

**Blocked calls never reach the tool server.** The agent receives an error response, and the event is logged as CRITICAL.

---

## Detection Engines

NEXUS runs four detection engines on every tool call:

### Injection Detector
Detects prompt injection in tool responses using 20+ patterns:
- Direct instruction overrides (`ignore previous instructions`)
- Role manipulation (`you are now DAN`)
- System prompt extraction attempts
- Hidden unicode characters
- HTML-embedded instructions (comments, hidden divs, data attributes, JSON-LD)
- Multi-language injection patterns

### DLP Engine (Data Loss Prevention)
Scans request and response content for sensitive data:

| Category | Patterns | Examples |
|----------|----------|---------|
| **PII** | 8 types | Email, phone, SSN, credit card, IP address |
| **Secrets** | 10+ types | AWS keys (`AKIA...`), Stripe (`sk_live_`), GitHub tokens, Slack tokens (`xoxb-`), private keys, JWTs, database connection strings |

Each finding includes confidence score, location, and pattern type.

### Data Exfiltration Detector
Identifies outbound data transfer attempts:
- Outbound tool detection (email, HTTP POST, file upload)
- Base64/hex encoded data in arguments
- Large data aggregation patterns
- Sensitive data in outbound payloads

### Policy Engine
Configurable pre-execution rules:
- **Sensitive path blocking** -- `.env`, `credentials`, `secrets`, `private_key`
- **Tool access control** -- Allow/deny specific tools per agent
- **Rate limiting** -- Throttle suspicious activity
- **Compliance-aware decisions** -- Flag PII processing for GDPR audit

Policy decisions: `ALLOW` | `ALLOW_WITH_AUDIT` | `MONITOR` | `BLOCK` | `REQUIRE_REVIEW`

---

## Dashboard

<p align="center">
  <img src="nexus2.png" alt="NEXUS Real-Time Monitoring" width="100%">
</p>

The NEXUS dashboard provides real-time visibility into all agent activity:

- **Live Activity Feed** -- WebSocket-powered real-time event stream
- **Threat Distribution** -- Visual breakdown of threat types and severity
- **Alert Management** -- Triage workflow (Open -> Investigating -> Resolved)
- **Agent Timeline** -- Per-agent activity with threat markers
- **Audit Trail** -- Immutable, hash-chained log with integrity verification

<p align="center">
  <img src="nexus3.png" alt="NEXUS Alerts & Audit" width="100%">
</p>

### Alert States

| State | Description |
|-------|-------------|
| **Open** | New alert, needs attention |
| **Acknowledged** | Team aware, not yet investigating |
| **Investigating** | Active investigation with notes |
| **Resolved** | Issue addressed with resolution notes |
| **False Positive** | Marked as non-threat |

---

## Compliance

NEXUS includes automated compliance assessment agents for **GDPR** and **NIS2** regulatory frameworks.

### GDPR Assessment
| Article | Focus | What It Checks |
|---------|-------|----------------|
| Article 5 | Processing Principles | Lawfulness, fairness, transparency, purpose limitation |
| Article 6 | Lawfulness | Legal basis validation for data processing |
| Article 7 | Consent | Consent mechanisms and documentation |
| Article 32 | Security | Technical and organizational security measures |
| Article 35 | DPIA | Data Protection Impact Assessment requirements |

### NIS2 Assessment
Validates all 10 minimum security measures:
- Risk analysis & information security policies
- Incident handling & detection procedures
- Business continuity & crisis management
- Supply chain security
- Network & information systems security
- Vulnerability management
- Effectiveness assessment
- Cyber hygiene & training
- Cryptography & key management
- Access control & identity management

### Compliance Reports
- Automated scoring (0-100%) per regulation
- Severity-based finding prioritization
- Remediation recommendations with deadlines
- PDF export for auditors
- Evidence collection and documentation

---

## Quick Start

### Prerequisites
- Java 21+
- Node.js 18+
- PostgreSQL 16
- Redis 7 (optional, for caching)

### 1. Clone and Build

```bash
git clone https://github.com/your-org/nexus.git
cd nexus

# Build all Java modules
mvn clean package -DskipTests

# Build dashboard
cd nexus-dashboard && npm install && npm run build && cd ..
```

### 2. Configure Database

```bash
# Create database
psql -U postgres -c "CREATE DATABASE nexus;"
psql -U postgres -c "CREATE USER nexus WITH PASSWORD 'nexus';"
psql -U postgres -c "GRANT ALL PRIVILEGES ON DATABASE nexus TO nexus;"
```

### 3. Start Services

```bash
# Start NEXUS Monitor (port 8080)
java -jar nexus-monitor/target/nexus-monitor-1.0.0-SNAPSHOT.jar

# Start Dashboard (port 3000)
cd nexus-dashboard && npm run preview

# (Optional) Start Compliance Agents (port 8081)
java -jar compliance-agents/target/compliance-agents-1.0.0-SNAPSHOT.jar
```

### 4. Point Your Agent at NEXUS

Configure your MCP-compatible agent to use `http://localhost:8080/mcp` as the tool server endpoint:

```yaml
# Example: agent configuration
mcp:
  server-url: http://localhost:8080/mcp  # NEXUS proxy, not the tool server directly
```

NEXUS auto-discovers tools from configured backends and exposes them to your agent transparently.

### 5. Configure Backends

Add your MCP tool servers to `nexus-monitor/src/main/resources/application.yml`:

```yaml
nexus:
  mcp:
    backends:
      - name: my-tools
        url: http://localhost:9000/mcp
        type: http
        healthCheck:
          enabled: true
          intervalSeconds: 30
```

---

## Module Overview

| Module | Port | Description |
|--------|------|-------------|
| **nexus-monitor** | 8080 | Core MCP proxy, threat detection, policy engine, audit trail |
| **nexus-dashboard** | 3000 | React dashboard with real-time monitoring |
| **compliance-agents** | 8081 | GDPR & NIS2 automated compliance assessment |
| **nexus-mcp-tools** | 8083 | Built-in MCP tools for compliance scanning |
| **nexus-common** | -- | Shared models and domain objects |

---

## Tech Stack

### Backend
- **Java 21** with virtual threads
- **Spring Boot 3.5** with WebSocket, Data JPA, Actuator
- **PostgreSQL 16** -- events, alerts, audit trail, compliance reports
- **Redis 7** -- caching, rate limiting, sessions
- **Google Cloud Pub/Sub** -- event streaming (optional)
- **Flyway** -- database migrations
- **Kotlin** -- compliance agents module

### Frontend
- **React 18** with TypeScript
- **Vite 5** -- build tooling
- **Tailwind CSS** + **shadcn/ui** -- styling
- **Recharts** -- data visualization
- **TanStack Query** -- server state management
- **Framer Motion** -- animations
- **WebSocket** -- real-time updates

---

## API Reference

### MCP Proxy
```
POST /mcp                          # MCP JSON-RPC 2.0 proxy endpoint
WS   /ws/events                    # WebSocket for real-time events
```

### Events
```
GET  /api/events                   # List events (paginated, filterable)
GET  /api/events/{eventId}         # Event details with threat analysis
POST /api/events/ingest            # Ingest non-MCP events
```

### Alerts
```
GET  /api/alerts                   # List alerts (filterable by severity, status)
GET  /api/alerts/{alertId}         # Alert details with evidence
POST /api/alerts/{id}/acknowledge  # Acknowledge alert
POST /api/alerts/{id}/investigate  # Start investigation
POST /api/alerts/{id}/resolve      # Resolve with notes
GET  /api/alerts/stats             # Alert statistics
```

### Audit Trail
```
GET  /api/audit/trail              # Hash-chained audit log
GET  /api/audit/compliance         # Compliance-relevant entries
GET  /api/audit/resource/{type}/{id}  # Resource-specific history
```

### Dashboard Metrics
```
GET  /api/dashboard/metrics        # KPI metrics (time-range aware)
GET  /api/dashboard/tools/top      # Top tools by usage
GET  /api/dashboard/alerts/categories  # Alert distribution
```

### Compliance
```
POST /api/compliance/check         # Start compliance assessment
GET  /api/compliance/reports       # List compliance reports
GET  /api/compliance/statistics    # Overall compliance scores
POST /api/compliance/reports/{id}/export/pdf  # Export report
```

Full API documentation available at `http://localhost:8080/swagger-ui.html` when running.

---

## Live Demo

The repository includes a complete demo showing a 4-step attack chain where a malicious website attempts to steal secrets through an AI agent, with NEXUS detecting every step:

| Step | Agent Action | NEXUS Response | Detection Engine |
|------|-------------|---------------|-----------------|
| 1 | `web_fetch` malicious page | Injection detected (CRITICAL) | InjectionDetector |
| 2 | `file_read .env` | **BLOCKED** before execution | PolicyEngine |
| 3 | `file_read internal-notes.txt` | Secrets flagged in response (HIGH) | DlpEngine |
| 4 | `send_mail` stolen secrets | Data exfiltration alert (CRITICAL) | ExfiltrationDetector + DLP |

```bash
# Run the demo
cd demos/yc-demo

# Start tool server (port 9001)
java -jar tool-server/target/yc-demo-tool-server-1.0.0-SNAPSHOT.jar &

# Start NEXUS (port 8080) -- must start after tool server
java -jar ../../nexus-monitor/target/nexus-monitor-1.0.0-SNAPSHOT.jar &

# Start agent (port 8082) -- must start after NEXUS
java -jar agent/target/yc-demo-agent-1.0.0-SNAPSHOT.jar &

# Open the demo UI
open http://localhost:8082

# Open the NEXUS dashboard
open http://localhost:3000
```

---

## Configuration Reference

```yaml
# nexus-monitor/src/main/resources/application.yml

server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/nexus
    username: nexus
    password: ${DB_PASSWORD:nexus}

  data:
    redis:
      cluster:
        nodes:
          - localhost:7000
          - localhost:7001
          - localhost:7002

nexus:
  mcp:
    backends:
      - name: my-tools
        url: http://localhost:9000/mcp
        type: http
        healthCheck:
          enabled: true
          intervalSeconds: 30

  pubsub:
    enabled: false  # Enable for Google Cloud Pub/Sub
    project-id: nexus-compliance
    topics:
      - nexus-agent-events
      - nexus-security-alerts
      - nexus-audit-entries

  detection:
    injection:
      enabled: true
    dlp:
      enabled: true
    exfiltration:
      enabled: true
```

---

## Contributing

We welcome contributions! Whether it's new detection patterns, policy rules, compliance frameworks, or dashboard improvements.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-detector`)
3. Commit your changes
4. Push to the branch (`git push origin feature/new-detector`)
5. Open a Pull Request

### Areas We Need Help With
- Additional injection detection patterns
- New compliance frameworks (SOC 2, HIPAA, ISO 27001)
- MCP transport support (stdio, SSE)
- MCP agent integrations (LangChain, CrewAI, AutoGen)
- Performance optimization for high-throughput deployments

---

## License

This project is licensed under the Apache License 2.0 -- see the [LICENSE](LICENSE) file for details.

---

<p align="center">
  <strong>NEXUS</strong> -- Because AI agents need a firewall too.
</p>
