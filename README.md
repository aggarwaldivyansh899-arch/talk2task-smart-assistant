# talk2task-smart-assistant
**talk2task-ai** is a futuristic AI productivity assistant for managing habits, tasks, meetings, reminders, and goals. Built with HTML, CSS, and JavaScript, it features an interactive chatbot, habit tracking, streaks, achievements, progress charts, and voice input. Powered by Google Gemini API for intelligent, conversational assistance.
[Uploading index (1).html…]()
<!DOCTYPE html>
<html lang="en">
<head>
<!-- ===== STEP 1: PASTE YOUR API KEY BETWEEN THE QUOTES ON LINE 6 ===== -->
<script>
const API_KEY = "PASTE_YOUR_API_KEY_HERE";
const MODEL = "gemini-3.5-flash-lite";
</script>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>talk2task</title>
<style>
*{box-sizing:border-box}body{margin:0;font:15px/1.55 system-ui,-apple-system,"Segoe UI",Roboto,sans-serif;background:radial-gradient(700px 400px at 80% -10%,#3b1d6e,transparent),#0e0e14;color:#eee;height:100vh;display:flex;justify-content:center}
.app{width:100%;max-width:760px;display:flex;flex-direction:column;height:100vh}
header{display:flex;align-items:center;gap:10px;padding:16px 18px;border-bottom:1px solid #2a2a38;font-weight:600;letter-spacing:.03em}
header i{width:10px;height:10px;border-radius:50%;background:#a855f7;box-shadow:0 0 12px #a855f7}header span{flex:1}
header button{background:none;border:1px solid #3a3a4a;color:#ccc;border-radius:8px;padding:5px 10px;cursor:pointer}
#log{flex:1;overflow-y:auto;padding:18px;display:flex;flex-direction:column;gap:12px}
.m{max-width:82%;padding:10px 14px;border-radius:14px;white-space:pre-wrap;word-wrap:break-word}
.ai{background:#1d1a2b;align-self:flex-start;border-bottom-left-radius:4px}
.me{background:linear-gradient(120deg,#a855f7,#6366f1);align-self:flex-end;border-bottom-right-radius:4px;color:#fff}
.err{background:#3a1515;border:1px solid #f87171;color:#fecaca;align-self:flex-start}
.dots span{display:inline-block;width:7px;height:7px;margin:0 2px;border-radius:50%;background:#a855f7;animation:d 1.1s infinite}.dots span:nth-child(2){animation-delay:.15s}.dots span:nth-child(3){animation-delay:.3s}
@keyframes d{0%,80%,100%{opacity:.2}40%{opacity:1}}
form{display:flex;gap:8px;padding:14px 16px 18px;border-top:1px solid #2a2a38}
textarea{flex:1;resize:none;height:46px;background:#16161f;border:1px solid #33334a;border-radius:12px;color:#eee;padding:12px;font:inherit}
form button{border:0;border-radius:12px;padding:0 18px;font-weight:600;color:#fff;background:linear-gradient(120deg,#a855f7,#6366f1);cursor:pointer}
form button:disabled{opacity:.5}
</style>
</head>
<body>
<div class="app">
  <header><i></i><span>talk2task</span><button id="clear" type="button">Clear chat</button></header>
  <div id="log"></div>
  <form id="f"><textarea id="q" placeholder="Ask a question…"></textarea><button id="send">Send</button></form>
</div>
<script>
// ===== YOUR CHATBOT'S INSTRUCTIONS (RTCC) =====
const SYSTEM_PROMPT = "ROLE: You are an expert AI developer and product designer.\nTASK: Build an AI Meeting Minutes Assistant that records/uploads meeting audio, converts it to text, summarizes the discussion, extracts decisions and action items, assigns tasks, and identifies deadlines.\nCONTEXT: The app should help users save time by automatically creating organized meeting minutes from conversations.\nCONSTRAINTS: Keep the UI simple, protect user data, avoid making up information, and mark unknown assignees or deadlines as “Not specified.”\nOUTPUT: Provide the project architecture, tech stack, main features, database structure, API flow, UI design, and MVP development plan.\nLANGUAGE: If the user writes in Hindi or Hinglish, reply in the same language.";

// ===== API CONNECTION (don't change) =====
async function callAI(history) {
  // history = [{ role: "user" | "assistant", text: "..." }, ...]
  const url = "https://generativelanguage.googleapis.com/v1beta/models/" + MODEL + ":generateContent";
  const res = await fetch(url, {
    method: "POST",
    headers: { "Content-Type": "application/json", "x-goog-api-key": API_KEY.trim() },
    body: JSON.stringify({
      system_instruction: { parts: [{ text: SYSTEM_PROMPT }] },
      contents: history.map(m => ({
        role: m.role === "assistant" ? "model" : "user",
        parts: [{ text: m.text }]
      }))
    })
  });
  const data = await res.json().catch(() => ({}));
  if (!res.ok) throw new Error("Error " + res.status + ": " + ((data.error && data.error.message) || "Request failed"));
  const parts = (data.candidates && data.candidates[0] && data.candidates[0].content && data.candidates[0].content.parts) || [];
  const text = parts.filter(p => p.text && !p.thought).map(p => p.text).join("");
  if (!text) throw new Error("The AI sent an empty reply. Try asking again.");
  return text;
}

// ===== CHAT =====
const log = document.getElementById("log"), q = document.getElementById("q"), send = document.getElementById("send");
let history = [];
function add(text, cls) { const d = document.createElement("div"); d.className = "m " + cls; d.textContent = text; log.appendChild(d); log.scrollTop = log.scrollHeight; return d; }
function welcome() { add("Hi! I'm talk2task. Ask me anything.", "ai"); }
welcome();
document.getElementById("clear").onclick = () => { history = []; log.innerHTML = ""; welcome(); };
q.addEventListener("keydown", e => { if (e.key === "Enter" && !e.shiftKey) { e.preventDefault(); document.getElementById("f").requestSubmit(); } });
document.getElementById("f").addEventListener("submit", async e => {
  e.preventDefault();
  const text = q.value.trim(); if (!text) return;
  add(text, "me"); q.value = "";
  if (!API_KEY || API_KEY === "PASTE_YOUR_API_KEY_HERE") { add("Paste your API key on line 6 of index.html, save, and refresh.", "err"); return; }
  history.push({ role: "user", text });
  send.disabled = true;
  const t = add("", "ai"); t.innerHTML = '<span class="dots"><span></span><span></span><span></span></span>';
  try { const reply = await callAI(history); history.push({ role: "assistant", text: reply }); t.textContent = reply; }
  catch (err) { history.pop(); t.className = "m err"; t.textContent = err.message; }
  send.disabled = false; q.focus(); log.scrollTop = log.scrollHeight;
});
</script>
</body>
</html>
