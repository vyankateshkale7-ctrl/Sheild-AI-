import { useState } from "react";

const C = {
  bg: "#060d1a",
  panel: "#0b1220",
  card: "#0f1830",
  border: "#1e2d4a",
  accent: "#00c6ff",
  accentDim: "#00c6ff18",
  accent2: "#7c3aed",
  safe: "#00e676",
  warn: "#ffb300",
  danger: "#ff4560",
  critical: "#ff0055",
  text: "#e8f0fe",
  muted: "#4a6080",
  sub: "#8a9bb5",
};

const TABS = [
  { icon: "🔍", label: "Phishing" },
  { icon: "🎭", label: "Deepfake" },
  { icon: "🔑", label: "Password" },
  { icon: "📊", label: "Privacy Score" },
  { icon: "🚨", label: "Threat Feed" },
  { icon: "🌐", label: "URL Scanner" },
  { icon: "💸", label: "UPI Scam" },
  { icon: "🕵️", label: "Dark Web" },
  { icon: "📧", label: "Email Header" },
];

const callClaude = async (prompt) => {
  const res = await fetch("

https://api.anthropic.com/v1/messages

", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      model: "claude-sonnet-4-6",
      max_tokens: 1000,
      messages: [{ role: "user", content: prompt }],
    }),
  });
  const data = await res.json();
  return data.content?.map(b => b.text || "").join("") || "";
};

const parseJSON = (raw) => {
  try { return JSON.parse(raw.replace(/```json|```/g, "").trim()); }
  catch { return null; }
};

const riskColor = (r) => ({ LOW: C.safe, MEDIUM: C.warn, HIGH: C.danger, CRITICAL: C.critical, SAFE: C.safe }[r] || C.muted);

const Btn = ({ onClick, disabled, children }) => (
  <button onClick={onClick} disabled={disabled} style={{
    marginTop: 14, width: "100%", padding: "13px 0", borderRadius: 10,
    background: disabled ? C.border : "linear-gradient(135deg, #00c6ff 0%, #7c3aed 100%)",
    color: disabled ? C.muted : "#fff", fontWeight: 700, fontSize: 15,
    border: "none", cursor: disabled ? "not-allowed" : "pointer", letterSpacing: 0.4,
    transition: "opacity 0.2s", opacity: disabled ? 0.6 : 1,
  }}>{children}</button>
);

const Tag = ({ color, children }) => (
  <span style={{ background: color + "22", color, borderRadius: 20, padding: "3px 12px", fontSize: 12, fontWeight: 700 }}>{children}</span>
);

const InfoBox = ({ color = C.accent, label, children }) => (
  <div style={{ background: color + "12", border: `1px solid ${color}40`, borderRadius: 8, padding: 14, marginTop: 10 }}>
    <div style={{ color, fontSize: 11, fontWeight: 700, marginBottom: 5 }}>{label}</div>
    <div style={{ color: C.text, fontSize: 14 }}>{children}</div>
  </div>
);

const ResultCard = ({ color, children }) => (
  <div style={{ marginTop: 18, background: C.bg, border: `1px solid ${color}`, borderRadius: 14, padding: 20 }}>{children}</div>
);

const TextArea = ({ value, onChange, placeholder }) => (
  <textarea value={value} onChange={onChange} placeholder={placeholder} style={{
    width: "100%", minHeight: 110, background: C.bg, border: `1px solid ${C.border}`,
    color: C.text, borderRadius: 8, padding: 12, fontSize: 14, resize: "vertical",
    fontFamily: "monospace", boxSizing: "border-box", outline: "none", lineHeight: 1.6,
  }} />
);

const Input = ({ value, onChange, placeholder, type = "text" }) => (
  <input type={type} value={value} onChange={onChange} placeholder={placeholder} style={{
    width: "100%", background: C.bg, border: `1px solid ${C.border}`,
    color: C.text, borderRadius: 8, padding: "11px 14px", fontSize: 14,
    outline: "none", boxSizing: "border-box",
  }} />
);

const FlagList = ({ items, color }) => items?.length > 0 && (
  <div style={{ marginBottom: 12 }}>
    {items.map((f, i) => <div key={i} style={{ color, fontSize: 13, marginBottom: 5 }}>▸ {f}</div>)}
  </div>
);

const SectionLabel = ({ children }) => (
  <div style={{ color: C.muted, fontSize: 11, fontWeight: 700, letterSpacing: 1, marginBottom: 6, marginTop: 14 }}>{children}</div>
);

// ── 1. Phishing Detector ───────────────────────────────────────────
function PhishingDetector() {
  const [input, setInput] = useState("");
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  const analyze = async () => {
    setLoading(true); setResult(null);
    const r = parseJSON(await callClaude(`Cybersecurity expert. Analyze for phishing/scam/social engineering. Text: "${input}"
JSON only (no markdown): {"risk":"LOW"|"MEDIUM"|"HIGH"|"CRITICAL","score":0-100,"verdict":"string","red_flags":["..."],"recommendation":"string"}`));
    setResult(r || { risk: "ERROR", verdict: "Analysis failed.", red_flags: [], recommendation: "Try again." });
    setLoading(false);
  };

  return (
    <div>
      <p style={{ color: C.sub, fontSize: 13, marginBottom: 14 }}>Paste any suspicious email, SMS, link, or message below.</p>
      <TextArea value={input} onChange={e => setInput(e.target.value)} placeholder="Paste suspicious content here..." />
      <Btn onClick={analyze} disabled={loading || !input.trim()}>{loading ? "⏳ Analyzing..." : "🔍 Analyze for Phishing"}</Btn>
      {result && (
        <ResultCard color={riskColor(result.risk)}>
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 14 }}>
            <div>
              <Tag color={riskColor(result.risk)}>{result.risk} RISK</Tag>
              <div style={{ color: C.text, marginTop: 8, fontSize: 15 }}>{result.verdict}</div>
            </div>
            {result.score !== undefined && (
              <div style={{ textAlign: "center", minWidth: 60 }}>
                <div style={{ color: riskColor(result.risk), fontSize: 30, fontWeight: 900 }}>{result.score}</div>
                <div style={{ color: C.muted, fontSize: 11 }}>/ 100</div>
              </div>
            )}
          </div>
          <SectionLabel>RED FLAGS</SectionLabel>
          <FlagList items={result.red_flags} color={C.warn} />
          <InfoBox label="WHAT TO DO">{result.recommendation}</InfoBox>
        </ResultCard>
      )}
    </div>
  );
}

// ── 2. Deepfake Alert ──────────────────────────────────────────────
function DeepfakeAlert() {
  const [input, setInput] = useState("");
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  const analyze = async () => {
    setLoading(true); setResult(null);
    const r = parseJSON(await callClaude(`Deepfake & AI manipulation detection expert. Analyze: "${input}"
JSON only: {"likelihood":"LOW"|"MEDIUM"|"HIGH","verdict":"string","warning_signs":["..."],"verification_steps":["..."],"tip":"string"}`));
    setResult(r || { likelihood: "UNKNOWN", verdict: "Failed.", warning_signs: [], verification_steps: [], tip: "Verify through known channels." });
    setLoading(false);
  };

  return (
    <div>
      <p style={{ color: C.sub, fontSize: 13, marginBottom: 14 }}>Describe a suspicious call, video, or voice message you received.</p>
      <TextArea value={input} onChange={e => setInput(e.target.value)} placeholder='E.g. "My boss called via video asking me to urgently transfer ₹50,000. His voice sounded robotic..."' />
      <Btn onClick={analyze} disabled={loading || !input.trim()}>{loading ? "⏳ Analyzing..." : "🎭 Check for Deepfake"}</Btn>
      {result && (
        <ResultCard color={riskColor(result.likelihood)}>
          <Tag color={riskColor(result.likelihood)}>{result.likelihood} DEEPFAKE LIKELIHOOD</Tag>
          <div style={{ color: C.text, marginTop: 10, marginBottom: 14 }}>{result.verdict}</div>
          <SectionLabel>WARNING SIGNS</SectionLabel>
          <FlagList items={result.warning_signs} color={C.warn} />
          <SectionLabel>VERIFICATION STEPS</SectionLabel>
          <FlagList items={result.verification_steps} color={C.safe} />
          <InfoBox label="PRO TIP">{result.tip}</InfoBox>
        </ResultCard>
      )}
    </div>
  );
}

// ── 3. Password Checker ────────────────────────────────────────────
function PasswordChecker() {
  const [pwd, setPwd] = useState("");
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  const analyze = async () => {
    setLoading(true); setResult(null);
    const r = parseJSON(await callClaude(`Password security expert. Analyze: length=${pwd.length}, upper=${/[A-Z]/.test(pwd)}, lower=${/[a-z]/.test(pwd)}, num=${/[0-9]/.test(pwd)}, sym=${/[^A-Za-z0-9]/.test(pwd)}, common=${/(123|abc|pass|qwerty|admin)/i.test(pwd)}
JSON only: {"strength":"VERY WEAK"|"WEAK"|"MODERATE"|"STRONG"|"VERY STRONG","score":0-100,"crack_time":"string","issues":["..."],"improvements":["..."],"suggested_pattern":"string"}`));
    setResult(r || { strength: "ERROR", score: 0, issues: ["Analysis failed."], improvements: [], crack_time: "Unknown" });
    setLoading(false);
  };

  const sc = { "VERY WEAK": C.critical, WEAK: C.danger, MODERATE: C.warn, STRONG: "#69f0ae", "VERY STRONG": C.safe };

  return (
    <div>
      <p style={{ color: C.sub, fontSize: 13, marginBottom: 14 }}>Test your password. It's analyzed privately — never stored.</p>
      <Input type="password" value={pwd} onChange={e => setPwd(e.target.value)} placeholder="Enter password to check strength..." />
      {pwd && (
        <div style={{ marginTop: 8, background: C.border, borderRadius: 4, height: 5 }}>
          <div style={{ height: 5, borderRadius: 4, background: pwd.length < 6 ? C.danger : pwd.length < 10 ? C.warn : C.safe, width: `${Math.min(100, pwd.length * 6.5)}%`, transition: "width 0.3s" }} />
        </div>
      )}
      <Btn onClick={analyze} disabled={loading || !pwd.trim()}>{loading ? "⏳ Checking..." : "🔑 Check Password"}</Btn>
      {result && (
        <ResultCard color={sc[result.strength] || C.muted}>
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 14 }}>
            <div>
              <Tag color={sc[result.strength] || C.muted}>{result.strength}</Tag>
              <div style={{ color: C.sub, fontSize: 13, marginTop: 6 }}>Crack time: <span style={{ color: C.text }}>{result.crack_time}</span></div>
            </div>
            <div style={{ textAlign: "center" }}>
              <div style={{ color: sc[result.strength] || C.muted, fontSize: 32, fontWeight: 900 }}>{result.score}</div>
              <div style={{ color: C.muted, fontSize: 11 }}>/ 100</div>
            </div>
          </div>
          <SectionLabel>ISSUES</SectionLabel>
          <FlagList items={result.issues} color={C.danger} />
          <SectionLabel>HOW TO IMPROVE</SectionLabel>
          <FlagList items={result.improvements} color={C.safe} />
          {result.suggested_pattern && <InfoBox label="BETTER PATTERN">{result.suggested_pattern}</InfoBox>}
        </ResultCard>
      )}
    </div>
  );
}

// ── 4. Privacy Score ───────────────────────────────────────────────
function PrivacyScore() {
  const qs = [
    { q: "Do you reuse passwords across websites?", k: "reuse" },
    { q: "Do you use 2-Factor Authentication (2FA)?", k: "tfa" },
    { q: "Do you use a VPN regularly?", k: "vpn" },
    { q: "Do you share personal info freely on social media?", k: "social" },
    { q: "Do you use public WiFi without protection?", k: "wifi" },
    { q: "Do you keep devices & apps updated?", k: "update" },
    { q: "Do you use encrypted messaging (Signal etc.)?", k: "encrypt" },
    { q: "Have you checked if your email was in a data breach?", k: "breach" },
  ];
  const [ans, setAns] = useState({});
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  const go = async () => {
    setLoading(true); setResult(null);
    const summary = qs.map(q => `${q.q} → ${ans[q.k]}`).join("\n");
    const r = parseJSON(await callClaude(`Privacy expert. Score user based on:\n${summary}\nJSON only: {"score":0-100,"grade":"A"|"B"|"C"|"D"|"F","level":"Excellent"|"Good"|"Fair"|"Poor"|"Critical","top_risks":["..."],"quick_wins":["..."],"summary":"2 sentences"}`));
    setResult(r || { score: 0, grade: "?", level: "Error", top_risks: [], quick_wins: [], summary: "Try again." });
    setLoading(false);
  };

  const gc = { A: C.safe, B: "#69f0ae", C: C.warn, D: C.danger, F: C.critical };

  return (
    <div>
      <p style={{ color: C.sub, fontSize: 13, marginBottom: 16 }}>Answer 8 quick questions to see your privacy grade.</p>
      {qs.map(({ q, k }) => (
        <div key={k} style={{ marginBottom: 14 }}>
          <div style={{ color: C.text, fontSize: 13, marginBottom: 7 }}>{q}</div>
          <div style={{ display: "flex", gap: 7 }}>
            {["Yes", "No", "Sometimes"].map(o => (
              <button key={o} onClick={() => setAns(p => ({ ...p, [k]: o }))} style={{
                padding: "6px 16px", borderRadius: 20, fontSize: 12, cursor: "pointer",
                border: `1px solid ${ans[k] === o ? C.accent : C.border}`,
                background: ans[k] === o ? C.accentDim : C.bg,
                color: ans[k] === o ? C.accent : C.muted,
              }}>{o}</button>
            ))}
          </div>
        </div>
      ))}
      <Btn onClick={go} disabled={loading || qs.some(q => !ans[q.k])}>{loading ? "⏳ Calculating..." : "📊 Get My Privacy Score"}</Btn>
      {result && (
        <ResultCard color={gc[result.grade] || C.muted}>
          <div style={{ display: "flex", alignItems: "center", gap: 20, marginBottom: 16 }}>
            <div style={{ fontSize: 60, color: gc[result.grade], fontWeight: 900, lineHeight: 1 }}>{result.grade}</div>
            <div>
              <div style={{ color: gc[result.grade], fontSize: 22, fontWeight: 700 }}>{result.score}/100</div>
              <div style={{ color: C.text }}>{result.level} Privacy</div>
              <div style={{ color: C.sub, fontSize: 13, marginTop: 4 }}>{result.summary}</div>
            </div>
          </div>
          <SectionLabel>TOP RISKS</SectionLabel>
          <FlagList items={result.top_risks} color={C.danger} />
          <SectionLabel>DO THESE TODAY</SectionLabel>
          <FlagList items={result.quick_wins} color={C.safe} />
        </ResultCard>
      )}
    </div>
  );
}

// ── 5. Threat Feed ─────────────────────────────────────────────────
function ThreatFeed() {
  const [region, setRegion] = useState("India");
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);
  const regions = ["India", "Pune/Maharashtra", "Global", "South Asia", "Europe", "USA"];
  const lc = { CRITICAL: C.critical, HIGH: C.danger, MEDIUM: C.warn };

  const go = async () => {
    setLoading(true); setResult(null);
    const r = parseJSON(await callClaude(`Threat intelligence analyst. Current 2026 threat briefing for ${region}.
JSON only: {"alerts":[{"level":"CRITICAL"|"HIGH"|"MEDIUM","title":"string","description":"string","action":"string"},{"level":"...","title":"...","description":"...","action":"..."},{"level":"...","title":"...","description":"...","action":"..."},{"level":"...","title":"...","description":"...","action":"..."}],"tip_of_day":"string","safe_tools":["...","...","..."]}`));
    setResult(r || { alerts: [], tip_of_day: "Stay vigilant.", safe_tools: [] });
    setLoading(false);
  };

  return (
    <div>
      <p style={{ color: C.sub, fontSize: 13, marginBottom: 14 }}>Get a live AI threat briefing for your region.</p>
      <div style={{ display: "flex", flexWrap: "wrap", gap: 7, marginBottom: 14 }}>
        {regions.map(r => (
          <button key={r} onClick={() => setRegion(r)} style={{
            padding: "6px 14px", borderRadius: 20, fontSize: 12, cursor: "pointer",
            border: `1px solid ${region === r ? C.accent : C.border}`,
            background: region === r ? C.accentDim : C.bg,
            color: region === r ? C.accent : C.muted,
          }}>{r}</button>
        ))}
      </div>
      <Btn onClick={go} disabled={loading}>{loading ? "⏳ Fetching..." : `🚨 Get ${region} Threat Feed`}</Btn>
      {result && (
        <div style={{ marginTop: 18 }}>
          {result.alerts?.map((a, i) => (
            <div key={i} style={{ background: C.bg, border: `1px solid ${lc[a.level] || C.border}`, borderRadius: 10, padding: 16, marginBottom: 12 }}>
              <Tag color={lc[a.level] || C.muted}>{a.level}</Tag>
              <div style={{ color: C.text, fontWeight: 600, marginTop: 8, marginBottom: 5 }}>{a.title}</div>
              <div style={{ color: C.sub, fontSize: 13, marginBottom: 10 }}>{a.description}</div>
              <InfoBox label="ACTION" color={C.accent}>{a.action}</InfoBox>
            </div>
          ))}
          {result.tip_of_day && <InfoBox label="💡 TIP OF THE DAY" color={C.safe}>{result.tip_of_day}</InfoBox>}
          {result.safe_tools?.length > 0 && (
            <div style={{ marginTop: 12 }}>
              <SectionLabel>RECOMMENDED TOOLS</SectionLabel>
              <div style={{ display: "flex", flexWrap: "wrap", gap: 7 }}>
                {result.safe_tools.map((t, i) => <Tag key={i} color={C.accent}>{t}</Tag>)}
              </div>
            </div>
          )}
        </div>
      )}
    </div>
  );
}

// ── 6. URL Scanner ─────────────────────────────────────────────────
function URLScanner() {
  const [url, setUrl] = useState("");
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  const analyze = async () => {
    setLoading(true); setResult(null);
    const r = parseJSON(await callClaude(`Cybersecurity URL analyst. Analyze this URL for malware, phishing, scams, suspicious domains: "${url}"
JSON only: {"risk":"SAFE"|"LOW"|"MEDIUM"|"HIGH"|"CRITICAL","score":0-100,"verdict":"string","red_flags":["..."],"domain_analysis":"string","recommendation":"string","safe_to_visit":true|false}`));
    setResult(r || { risk: "ERROR", verdict: "Analysis failed.", red_flags: [], recommendation: "Do not visit this URL.", safe_to_visit: false });
    setLoading(false);
  };

  return (
    <div>
      <p style={{ color: C.sub, fontSize: 13, marginBottom: 14 }}>Scan any URL or website for malware, phishing, and scams before visiting.</p>
      <Input value={url} onChange={e => setUrl(e.target.value)} placeholder="
https://suspicious-website.com
" />
      <Btn onClick={analyze} disabled={loading || !url.trim()}>{loading ? "⏳ Scanning..." : "🌐 Scan URL"}</Btn>
      {result && (
        <ResultCard color={riskColor(result.risk)}>
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 14 }}>
            <div>
              <Tag color={riskColor(result.risk)}>{result.risk}</Tag>
              <div style={{ marginTop: 8 }}>
                <span style={{ fontSize: 18 }}>{result.safe_to_visit ? "✅" : "🚫"}</span>
                <span style={{ color: result.safe_to_visit ? C.safe : C.danger, fontWeight: 700, marginLeft: 8 }}>
                  {result.safe_to_visit ? "Safe to Visit" : "DO NOT VISIT"}
                </span>
              </div>
              <div style={{ color: C.text, fontSize: 14, marginTop: 6 }}>{result.verdict}</div>
            </div>
            {result.score !== undefined && (
              <div style={{ textAlign: "center" }}>
                <div style={{ color: riskColor(result.risk), fontSize: 30, fontWeight: 900 }}>{result.score}</div>
                <div style={{ color: C.muted, fontSize: 11 }}>THREAT</div>
              </div>
            )}
          </div>
          {result.domain_analysis && <InfoBox label="DOMAIN ANALYSIS" color={C.accent2}>{result.domain_analysis}</InfoBox>}
          <SectionLabel>RED FLAGS</SectionLabel>
          <FlagList items={result.red_flags} color={C.warn} />
          <InfoBox label="RECOMMENDATION">{result.recommendation}</InfoBox>
        </ResultCard>
      )}
    </div>
  );
}

// ── 7. UPI Scam Detector ───────────────────────────────────────────
function UPIScamDetector() {
  const [input, setInput] = useState("");
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  const analyze = async () => {
    setLoading(true); setResult(null);
    const r = parseJSON(await callClaude(`India UPI/digital payment scam expert. Analyze this scenario for fraud: "${input}"
Common Indian scams: KYC fraud, refund scam, lottery, screen sharing, fake customer care, QR code scam, job fraud, OTP theft.
JSON only: {"risk":"LOW"|"MEDIUM"|"HIGH"|"CRITICAL","scam_type":"string or null","verdict":"string","red_flags":["..."],"never_do":["..."],"what_to_do":"string","report_to":"string"}`));
    setResult(r || { risk: "ERROR", verdict: "Analysis failed.", red_flags: [], never_do: [], what_to_do: "Never share OTP or PIN.", report_to: "cybercrime.gov.in" });
    setLoading(false);
  };

  return (
    <div>
      <p style={{ color: C.sub, fontSize: 13, marginBottom: 14 }}>Describe a suspicious UPI request, call, or payment scenario to check for fraud.</p>
      <TextArea value={input} onChange={e => setInput(e.target.value)} placeholder='E.g. "Someone called saying my GPay KYC expired and asked me to scan a QR code to verify..."' />
      <Btn onClick={analyze} disabled={loading || !input.trim()}>{loading ? "⏳ Analyzing..." : "💸 Check for UPI Scam"}</Btn>
      {result && (
        <ResultCard color={riskColor(result.risk)}>
          <div style={{ marginBottom: 14 }}>
            <Tag color={riskColor(result.risk)}>{result.risk} RISK</Tag>
            {result.scam_type && <Tag color={C.accent2}>{result.scam_type}</Tag>}
            <div style={{ color: C.text, marginTop: 10 }}>{result.verdict}</div>
          </div>
          <SectionLabel>RED FLAGS</SectionLabel>
          <FlagList items={result.red_flags} color={C.warn} />
          <SectionLabel>NEVER DO THIS</SectionLabel>
          <FlagList items={result.never_do} color={C.danger} />
          <InfoBox label="WHAT TO DO NOW">{result.what_to_do}</InfoBox>
          {result.report_to && <InfoBox label="REPORT TO" color={C.safe}>{result.report_to}</InfoBox>}
        </ResultCard>
      )}
    </div>
  );
}

// ── 8. Dark Web Monitor ────────────────────────────────────────────
function DarkWebMonitor() {
  const [email, setEmail] = useState("");
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  const check = async () => {
    setLoading(true); setResult(null);
    const domain = email.split("@")[1] || "";
    const r = parseJSON(await callClaude(`Dark web & data breach expert. User email domain: ${domain}, email pattern suggests: ${email.includes("gmail") ? "Gmail" : email.includes("yahoo") ? "Yahoo" : "custom domain"}.
Simulate a realistic breach check advisory (do NOT invent specific breach names falsely — give general guidance).
JSON only: {"breach_likelihood":"LOW"|"MEDIUM"|"HIGH","common_breaches_for_this_type":["..."],"exposed_data_types":["..."],"immediate_actions":["..."],"monitoring_tips":["..."],"tools":["..."],"summary":"string"}`));
    setResult(r || { breach_likelihood: "UNKNOWN", common_breaches_for_this_type: [], exposed_data_types: [], immediate_actions: ["Change your password immediately."], monitoring_tips: [], tools: ["haveibeenpwned.com"], summary: "Check manually at haveibeenpwned.com" });
    setLoading(false);
  };

  const lc = { LOW: C.safe, MEDIUM: C.warn, HIGH: C.danger, UNKNOWN: C.muted };

  return (
    <div>
      <p style={{ color: C.sub, fontSize: 13, marginBottom: 14 }}>Check if your email type is commonly found in dark web data breaches and get protection advice.</p>
      <div style={{ background: C.warn + "15", border: `1px solid ${C.warn}40`, borderRadius: 8, padding: 12, marginBottom: 14, fontSize: 12, color: C.warn }}>
        ⚠️ For real-time breach checking, visit <strong>haveibeenpwned.com</strong> directly. This tool provides advisory guidance.
      </div>
      <Input value={email} onChange={e => setEmail(e.target.value)} placeholder="yourname@gmail.com" />
      <Btn onClick={check} disabled={loading || !email.includes("@")}>{loading ? "⏳ Checking..." : "🕵️ Check Dark Web Exposure"}</Btn>
      {result && (
        <ResultCard color={lc[result.breach_likelihood]}>
          <Tag color={lc[result.breach_likelihood]}>{result.breach_likelihood} BREACH LIKELIHOOD</Tag>
          <div style={{ color: C.text, marginTop: 10, marginBottom: 14 }}>{result.summary}</div>
          <SectionLabel>COMMON BREACHES FOR THIS EMAIL TYPE</SectionLabel>
          <FlagList items={result.common_breaches_for_this_type} color={C.warn} />
          <SectionLabel>TYPES OF DATA OFTEN EXPOSED</SectionLabel>
          <FlagList items={result.exposed_data_types} color={C.danger} />
          <SectionLabel>IMMEDIATE ACTIONS</SectionLabel>
          <FlagList items={result.immediate_actions} color={C.safe} />
          <SectionLabel>MONITORING TOOLS</SectionLabel>
          <div style={{ display: "flex", flexWrap: "wrap", gap: 7, marginTop: 4 }}>
            {result.tools?.map((t, i) => <Tag key={i} color={C.accent}>{t}</Tag>)}
          </div>
        </ResultCard>
      )}
    </div>
  );
}

// ── 9. Email Header Analyzer ───────────────────────────────────────
function EmailHeaderAnalyzer() {
  const [header, setHeader] = useState("");
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  const analyze = async () => {
    setLoading(true); setResult(null);
    const r = parseJSON(await callClaude(`Email forensics expert. Analyze these email headers for spoofing, fake senders, suspicious routing: "${header.substring(0, 800)}"
JSON only: {"verdict":"LEGITIMATE"|"SUSPICIOUS"|"SPOOFED"|"UNKNOWN","risk":"LOW"|"MEDIUM"|"HIGH"|"CRITICAL","sender_analysis":"string","routing_analysis":"string","spf_dkim_status":"string","red_flags":["..."],"conclusion":"string"}`));
    setResult(r || { verdict: "UNKNOWN", risk: "MEDIUM", red_flags: [], conclusion: "Could not analyze headers." });
    setLoading(false);
  };

  const vc = { LEGITIMATE: C.safe, SUSPICIOUS: C.warn, SPOOFED: C.danger, UNKNOWN: C.muted };

  return (
    <div>
      <p style={{ color: C.sub, fontSize: 13, marginBottom: 8 }}>Paste raw email headers to detect spoofing and fake senders.</p>
      <div style={{ color: C.muted, fontSize: 12, marginBottom: 14 }}>
        In Gmail: Open email → ⋮ menu → "Show original" → copy headers
      </div>
      <TextArea value={header} onChange={e => setHeader(e.target.value)} placeholder="Paste raw email headers here...
Received: from mail.example.com...
From: someone@domain.com
To: you@gmail.com..." />
      <Btn onClick={analyze} disabled={loading || !header.trim()}>{loading ? "⏳ Analyzing headers..." : "📧 Analyze Email Headers"}</Btn>
      {result && (
        <ResultCard color={vc[result.verdict] || C.muted}>
          <div style={{ marginBottom: 14 }}>
            <Tag color={vc[result.verdict] || C.muted}>{result.verdict}</Tag>
            <span style={{ marginLeft: 8 }}><Tag color={riskColor(result.risk)}>{result.risk} RISK</Tag></span>
          </div>
          {result.sender_analysis && <InfoBox label="SENDER ANALYSIS" color={C.accent}>{result.sender_analysis}</InfoBox>}
          {result.routing_analysis && <InfoBox label="ROUTING ANALYSIS" color={C.accent2}>{result.routing_analysis}</InfoBox>}
          {result.spf_dkim_status && <InfoBox label="SPF / DKIM STATUS" color={C.warn}>{result.spf_dkim_status}</InfoBox>}
          <SectionLabel>RED FLAGS</SectionLabel>
          <FlagList items={result.red_flags} color={C.danger} />
          <InfoBox label="CONCLUSION" color={C.safe}>{result.conclusion}</InfoBox>
        </ResultCard>
      )}
    </div>
  );
}

// ── Main App ───────────────────────────────────────────────────────
const PANELS = [PhishingDetector, DeepfakeAlert, PasswordChecker, PrivacyScore, ThreatFeed, URLScanner, UPIScamDetector, DarkWebMonitor, EmailHeaderAnalyzer];

export default function ShieldAIPro() {
  const [tab, setTab] = useState(0);
  const Panel = PANELS[tab];

  return (
    <div style={{ minHeight: "100vh", background: C.bg, color: C.text, fontFamily: "'Inter','Segoe UI',sans-serif" }}>
      {/* Header */}
      <div style={{ background: C.panel, borderBottom: `1px solid ${C.border}`, padding: "18px 20px" }}>
        <div style={{ maxWidth: 720, margin: "0 auto", display: "flex", alignItems: "center", gap: 14 }}>
          <div style={{ fontSize: 32 }}>🛡️</div>
          <div>
            <div style={{ fontSize: 22, fontWeight: 900, letterSpacing: -0.5 }}>
              Shield<span style={{ color: C.accent }}>AI</span>
              <span style={{ color: C.accent2, fontSize: 13, fontWeight: 600, marginLeft: 8 }}>PRO</span>
            </div>
            <div style={{ color: C.muted, fontSize: 12 }}>9-in-1 AI Security Guardian · Made for India 🇮🇳</div>
          </div>
          <div style={{ marginLeft: "auto", background: C.safe + "20", color: C.safe, borderRadius: 20, padding: "4px 12px", fontSize: 11, fontWeight: 700 }}>
            ● ONLINE
          </div>
        </div>
      </div>

      {/* Tab bar */}
      <div style={{ background: C.panel, borderBottom: `1px solid ${C.border}`, overflowX: "auto" }}>
        <div style={{ maxWidth: 720, margin: "0 auto", display: "flex" }}>
          {TABS.map((t, i) => (
            <button key={i} onClick={() => setTab(i)} style={{
              padding: "11px 14px", background: "none", border: "none", cursor: "pointer",
              color: tab === i ? C.accent : C.muted,
              fontWeight: tab === i ? 700 : 400, fontSize: 12,
              borderBottom: `2px solid ${tab === i ? C.accent : "transparent"}`,
              whiteSpace: "nowrap", transition: "color 0.2s",
              display: "flex", flexDirection: "column", alignItems: "center", gap: 3,
            }}>
              <span style={{ fontSize: 16 }}>{t.icon}</span>
              <span>{t.label}</span>
            </button>
          ))}
        </div>
      </div>

      {/* Content */}
      <div style={{ maxWidth: 720, margin: "0 auto", padding: "20px 16px" }}>
        <div style={{ background: C.card, border: `1px solid ${C.border}`, borderRadius: 16, padding: 22 }}>
          <div style={{ color: C.accent, fontWeight: 800, fontSize: 15, marginBottom: 3 }}>
            {TABS[tab].icon} {TABS[tab].label}
          </div>
          <div style={{ borderBottom: `1px solid ${C.border}`, marginBottom: 18 }} />
          <Panel />
        </div>

        <div style={{ textAlign: "center", color: C.muted, fontSize: 11, marginTop: 20, lineHeight: 2 }}>
          🛡️ ShieldAI Pro · 9 AI Security Tools · Powered by Claude · Your data is never stored<br />
          Built for India 🇮🇳 · Report cybercrime at <span style={{ color: C.accent }}>cybercrime.gov.in</span> · Helpline: <span style={{ color: C.accent }}>1930</span>
        </div>
      </div>
    </div>
  );
}

