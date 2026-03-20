# DEBMM Self-Assessment Tool

An interactive, single-file web application for evaluating your detection engineering team's maturity against the **Elastic Detection Engineering Behavior Maturity Model (DEBMM)**. No server required, no database, no build step — open the HTML file in a browser and start assessing.

> Based on the [Elastic Security Labs DEBMM](https://www.elastic.co/security-labs/elastic-releases-debmm) published by Mika Ayenson, Terrance DeJesus, and Samir Bousseaden.

---

## What it does

The tool covers all five DEBMM maturity tiers across 16 criteria, and includes an integrated questionnaire, team metrics tracker, and AI-powered report generation.

### Assessment (Tiers 0–4)

- **16 criteria** across five tiers: Foundation, Basic, Intermediate, Advanced, Expert
- Score each criterion 0–4 (Initial → Optimized) using the DEBMM qualitative and quantitative descriptors
- Mark each score's **confidence level**: Self-assessed, Peer-reviewed, Data-evidenced, or Externally validated — scores are weighted accordingly (self-assessed high scores are downweighted by 20% in the adjusted total)
- Structured evidence fields per criterion: current evidence, gap to next level, owner, and target date
- Criteria blocking tier progression are flagged automatically
- All state persists to `localStorage` — close the tab and come back later

### Dashboard

- Live **confidence-adjusted score** and **raw self-assessed score**
- Current maturity tier with calculation explanation
- Per-tier progress bar with completion indicators
- Assessment **snapshot history** — save dated snapshots and track score movement over time with delta arrows
- Session metadata: team name, assessor, ruleset scope, assessment date

### Results & Analysis

- **Coverage heat map** — all 16 criteria colour-coded by level at a glance
- **Maturity radar chart** — tier averages visualised
- **Criteria distribution histogram** — how many criteria sit at each level
- Tier-by-tier breakdown with pass/fail status and per-criterion detail
- **Actionable recommendations** — criterion-specific, referencing the exact quantitative threshold for the next level, flagging evidence gaps on high self-assessed scores
- Print/PDF mode via browser print

### Team Metrics

- 9 editable KPI fields: alert volume, false positive rate, MTTD, rules deployed, rules deprecated, ATT&CK coverage %, false negative rate, hunting findings converted to rules, rules peer-reviewed %
- Each metric maps to the DEBMM criterion it evidences and shows T2/T3 thresholds
- **ATT&CK tactic coverage** — enter % coverage per tactic with colour-coded bars (red/amber/green by tier threshold)
- Expandable glossary of 12 key DEBMM terms

### Questionnaire

- 7 lifecycle phases: Identify Threat Landscape, Define the Perfect Rule, Define the Perfect Ruleset, Maintain, Test & Release, Criteria Assessment, Iterate
- **Interactive questions** — click to select answers (Yes/No/Partial etc), colour-coded green/amber by response
- **Checkable improvement steps** — mark each action complete as you work through it
- Per-phase progress bar showing answered questions and completed steps
- Cross-references to related DEBMM criteria — click to jump directly to the criterion card
- Phases auto-mark as reviewed when all questions are answered

### AI Report Generation

With an Anthropic API key configured, clicking **Export Report** generates a full professional maturity report via Claude Sonnet with nine sections:

1. Executive Summary — written for a CISO or board audience in plain language
2. Maturity Scorecard — all 5 tiers with score, status, strength, and biggest gap
3. Risk Assessment — threat categories undetected, operational risk narrative
4. Detailed Findings by Tier — every criterion with level, confidence, gap, and owner
5. Questionnaire Findings — what the answers reveal about current practices
6. Team Metrics Analysis — how KPIs validate or challenge self-assessed scores
7. Prioritised Improvement Roadmap — 0–90 days, 3–6 months, 6–12 months
8. KPIs to Track — table of 8–10 metrics with current value, T2 target, T3 target
9. Conclusion — one paragraph for management, one for the technical team

The report can be copied as Markdown or downloaded as a `.md` file. Without a key, a structured local fallback report is generated instead.

---

## Getting started

### Without AI report generation

1. Download `debmm-assessment.template.html`
2. Open it in any modern browser (Chrome, Firefox, Edge, Safari)
3. Start with Tier 0 and work through each criterion

That's it. No internet connection required after the Google Fonts load (which can be removed if needed).

### With AI report generation

The AI report feature calls the Anthropic API directly from the browser. Your API key must be set in the file — **keep this file in a private repository**.

**Step 1** — Get an API key from [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys)

**Step 2** — Copy the template and add your key

```bash
cp debmm-assessment.template.html debmm-assessment.html
```

Open `debmm-assessment.html` in a text editor and find the key variable near the top of the `<script>` block (around line 740):

```javascript
const ANTHROPIC_API_KEY = '';
```

Replace it with your key:

```javascript
const ANTHROPIC_API_KEY = 'sk-ant-api03-...';
```

**Step 3** — Open `debmm-assessment.html` in your browser

The working file (`debmm-assessment.html`) is in `.gitignore` and will never be committed. Only the template (with an empty key) is tracked by git.

---

## Repository structure

```
├── debmm-assessment.template.html   # ✓ Safe to commit — no API key
├── debmm-assessment.html            # ✗ Git-ignored — contains your key
├── .gitignore                       # Guards against accidental key commits
└── README.md
```

---

## Security

| File | Contains key? | Safe to commit publicly? |
|------|--------------|--------------------------|
| `debmm-assessment.template.html` | No (empty string) | Yes |
| `debmm-assessment.html` | Yes (your key) | **No — git-ignored** |
| `.gitignore` | No | Yes |
| `README.md` | No | Yes |

**Never paste your API key into the template file.** The `.gitignore` protects against accidental commits of the working file, but if you accidentally commit a real key anywhere, rotate it immediately at [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys).

The tool stores all assessment data in your browser's `localStorage` only. Nothing is sent to any server except the Anthropic API call when you click Generate Report — and that only sends your assessment context and the prompt, not any credentials or personal data beyond what you've typed into the tool.

For team use where multiple people need the AI report feature, see the [Proxy server option](#hosting-for-a-team) below.

---

## Hosting options

### Local use (simplest)

Open the HTML file directly in your browser. Works offline after initial font load.

### GitHub Pages (static, no AI)

1. Push the template file to a public or private repo
2. Go to **Settings → Pages → Deploy from branch**
3. Your tool is live at `https://yourusername.github.io/repo-name/debmm-assessment.template.html`

The template works fully without the API key — only the AI report generation is unavailable.

### Hosting for a team

If you want the AI report feature available to your whole team without each person managing their own key, run a small proxy server that holds the key server-side. The browser never sees the key.

**Cloudflare Workers (free tier, ~10 minutes to set up)**

```bash
npm install -g wrangler
wrangler login
mkdir debmm-proxy && cd debmm-proxy
wrangler init --no-bundle
```

`src/index.js`:

```javascript
export default {
  async fetch(request, env) {
    if (request.method === 'OPTIONS') {
      return new Response(null, { headers: corsHeaders() });
    }
    if (request.method !== 'POST' || new URL(request.url).pathname !== '/api/report') {
      return new Response('Not found', { status: 404 });
    }
    const body = await request.json();
    const upstream = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': env.ANTHROPIC_API_KEY,
        'anthropic-version': '2023-06-01'
      },
      body: JSON.stringify(body)
    });
    const data = await upstream.json();
    return new Response(JSON.stringify(data), {
      headers: { 'Content-Type': 'application/json', ...corsHeaders() }
    });
  }
};

function corsHeaders() {
  return {
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Methods': 'POST, OPTIONS',
    'Access-Control-Allow-Headers': 'Content-Type'
  };
}
```

```bash
wrangler secret put ANTHROPIC_API_KEY   # paste your key when prompted
wrangler deploy
```

Then in `debmm-assessment.html`, update the `exportReport` function to call your worker URL (`https://debmm-proxy.yourname.workers.dev/api/report`) instead of the Anthropic endpoint directly, and remove the `x-api-key` header from the client — the worker adds it.

---

## Cost estimate

Generating one AI report uses approximately 2,000 input tokens and 1,500 output tokens with Claude Sonnet. At current pricing this is under $0.01 per report. A team running 50 reports a month costs less than $0.50.

---

## Customising the tool

All content — tier names, criteria, qualitative descriptors, quantitative thresholds, questionnaire questions, improvement steps, metrics, glossary terms — is defined in the JavaScript data arrays at the top of the `<script>` block. Look for the `TIERS`, `METRICS`, `ATT_TACTICS`, `GLOSSARY`, `QUESTIONNAIRE`, and `APPLY_STEPS` constants. No framework or build step is needed — edit the arrays directly.

---

## Reference

- [Elastic DEBMM — Full model documentation](https://www.elastic.co/security-labs/elastic-releases-debmm)
- [Elastic Detection Rules (public)](https://github.com/elastic/detection-rules)
- [Detections as Code reference](https://dac-reference.readthedocs.io/en/latest/)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [SANS Threat Hunting Maturity Model](https://www.sans.org/tools/hunting-maturity-model/)
- [Anthropic API documentation](https://docs.anthropic.com)

---

## Licence

This tool is provided as-is for internal team use. The DEBMM model itself is the work of Elastic Security Labs — refer to the [original publication](https://www.elastic.co/security-labs/elastic-releases-debmm) for the authoritative model definition.
