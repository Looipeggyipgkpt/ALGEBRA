<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Study Buddy Bot 📚</title>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --bg: #f0f4ff;
      --surface: #ffffff;
      --primary: #4f46e5;
      --primary-dark: #4338ca;
      --primary-light: #eef2ff;
      --bot-bubble: #ffffff;
      --user-bubble: #4f46e5;
      --user-text: #ffffff;
      --bot-text: #1e293b;
      --muted: #64748b;
      --border: #e2e8f0;
      --danger: #ef4444;
      --success: #10b981;
      --font: -apple-system, "Segoe UI", system-ui, sans-serif;
    }

    body {
      font-family: var(--font);
      background: var(--bg);
      display: flex;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      padding: 12px;
    }

    .chat-container {
      width: 100%;
      max-width: 680px;
      height: 92vh;
      max-height: 800px;
      background: var(--surface);
      border-radius: 20px;
      box-shadow: 0 8px 40px rgba(0,0,0,0.12);
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }

    /* ── Header ── */
    .chat-header {
      background: var(--primary);
      color: #fff;
      padding: 16px 20px;
      display: flex;
      align-items: center;
      gap: 14px;
      flex-shrink: 0;
    }

    .avatar {
      width: 44px;
      height: 44px;
      background: rgba(255,255,255,0.2);
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 22px;
      flex-shrink: 0;
    }

    .header-info h1 {
      font-size: 17px;
      font-weight: 700;
      letter-spacing: 0.2px;
    }

    .header-info p {
      font-size: 12px;
      opacity: 0.82;
      margin-top: 2px;
    }

    .status-dot {
      width: 8px;
      height: 8px;
      background: #4ade80;
      border-radius: 50%;
      display: inline-block;
      margin-right: 5px;
    }

    /* ── Topic Pills ── */
    .topic-pills {
      padding: 10px 16px;
      display: flex;
      gap: 7px;
      flex-wrap: wrap;
      border-bottom: 1px solid var(--border);
      flex-shrink: 0;
      background: var(--primary-light);
    }

    .topic-pill {
      padding: 4px 12px;
      border-radius: 20px;
      border: 1.5px solid var(--primary);
      color: var(--primary);
      background: var(--surface);
      font-size: 11.5px;
      font-weight: 600;
      cursor: pointer;
      transition: all 0.15s;
      white-space: nowrap;
    }

    .topic-pill:hover {
      background: var(--primary);
      color: #fff;
    }

    /* ── Messages ── */
    .messages {
      flex: 1;
      overflow-y: auto;
      padding: 20px 16px;
      display: flex;
      flex-direction: column;
      gap: 10px;
      scroll-behavior: smooth;
    }

    .messages::-webkit-scrollbar { width: 5px; }
    .messages::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }

    .message-row {
      display: flex;
      align-items: flex-end;
      gap: 8px;
      animation: fadeIn 0.2s ease;
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(6px); }
      to   { opacity: 1; transform: translateY(0); }
    }

    .message-row.user { flex-direction: row-reverse; }

    .msg-avatar {
      width: 30px;
      height: 30px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 15px;
      flex-shrink: 0;
      background: var(--primary-light);
    }

    .message-row.user .msg-avatar {
      background: #e0e7ff;
    }

    .bubble {
      max-width: 78%;
      padding: 11px 15px;
      border-radius: 18px;
      font-size: 14px;
      line-height: 1.55;
      position: relative;
      word-break: break-word;
    }

    .message-row.bot .bubble {
      background: var(--bot-bubble);
      color: var(--bot-text);
      border: 1px solid var(--border);
      border-bottom-left-radius: 4px;
    }

    .message-row.user .bubble {
      background: var(--user-bubble);
      color: var(--user-text);
      border-bottom-right-radius: 4px;
    }

    .bubble.escalation {
      border-left: 3px solid var(--danger);
      background: #fff5f5;
      color: #7f1d1d;
    }

    /* Markdown-ish formatting */
    .bubble strong, .bubble b { font-weight: 700; }
    .bubble em { font-style: italic; }

    .timestamp {
      font-size: 10px;
      color: var(--muted);
      margin-top: 3px;
      padding: 0 6px;
    }

    .message-row.user .timestamp { text-align: right; }

    /* ── Typing Indicator ── */
    .typing-row { display: none; }
    .typing-row.active { display: flex; }

    .typing-bubble {
      background: var(--bot-bubble);
      border: 1px solid var(--border);
      border-radius: 18px;
      border-bottom-left-radius: 4px;
      padding: 11px 16px;
      display: flex;
      gap: 5px;
      align-items: center;
    }

    .dot {
      width: 7px; height: 7px;
      background: var(--muted);
      border-radius: 50%;
      animation: bounce 1.2s infinite;
    }
    .dot:nth-child(2) { animation-delay: 0.2s; }
    .dot:nth-child(3) { animation-delay: 0.4s; }

    @keyframes bounce {
      0%, 60%, 100% { transform: translateY(0); }
      30% { transform: translateY(-5px); }
    }

    /* ── Quick Replies ── */
    .quick-replies {
      padding: 8px 16px 4px;
      display: flex;
      gap: 7px;
      flex-wrap: wrap;
      flex-shrink: 0;
    }

    .qr-btn {
      padding: 6px 14px;
      border-radius: 20px;
      border: 1.5px solid var(--primary);
      color: var(--primary);
      background: var(--surface);
      font-size: 12.5px;
      font-weight: 600;
      cursor: pointer;
      transition: all 0.15s;
    }

    .qr-btn:hover {
      background: var(--primary);
      color: #fff;
    }

    /* ── Input Area ── */
    .input-area {
      padding: 12px 16px 16px;
      border-top: 1px solid var(--border);
      display: flex;
      gap: 10px;
      align-items: flex-end;
      flex-shrink: 0;
    }

    #msg-input {
      flex: 1;
      border: 1.5px solid var(--border);
      border-radius: 24px;
      padding: 11px 16px;
      font-size: 14px;
      font-family: var(--font);
      resize: none;
      outline: none;
      line-height: 1.4;
      max-height: 100px;
      overflow-y: auto;
      transition: border-color 0.15s;
      color: var(--bot-text);
    }

    #msg-input:focus {
      border-color: var(--primary);
    }

    #msg-input::placeholder { color: #94a3b8; }

    #send-btn {
      width: 44px;
      height: 44px;
      border-radius: 50%;
      background: var(--primary);
      border: none;
      color: #fff;
      font-size: 18px;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: background 0.15s, transform 0.1s;
      flex-shrink: 0;
    }

    #send-btn:hover { background: var(--primary-dark); }
    #send-btn:active { transform: scale(0.94); }
    #send-btn:disabled { background: #c7d2fe; cursor: not-allowed; }

    /* ── Footer ── */
    .chat-footer {
      padding: 6px 16px 10px;
      text-align: center;
      font-size: 11px;
      color: var(--muted);
      flex-shrink: 0;
    }

    /* ── Responsive ── */
    @media (max-width: 480px) {
      body { padding: 0; }
      .chat-container { height: 100vh; max-height: 100vh; border-radius: 0; }
      .bubble { max-width: 88%; }
    }
  </style>
</head>
<body>
  <div class="chat-container">
    <!-- Header -->
    <div class="chat-header">
      <div class="avatar">📚</div>
      <div class="header-info">
        <h1>Study Buddy Bot</h1>
        <p><span class="status-dot"></span>Online · Maths Tutor for Gen Z</p>
      </div>
    </div>

    <!-- Topic quick-access pills -->
    <div class="topic-pills">
      <button class="topic-pill" data-msg="Explain algebra to me">Algebra</button>
      <button class="topic-pill" data-msg="Explain geometry to me">Geometry</button>
      <button class="topic-pill" data-msg="Explain trigonometry">Trigonometry</button>
      <button class="topic-pill" data-msg="Explain calculus">Calculus</button>
      <button class="topic-pill" data-msg="Explain statistics">Statistics</button>
      <button class="topic-pill" data-msg="Explain probability">Probability</button>
      <button class="topic-pill" data-msg="Explain fractions">Fractions</button>
    </div>

    <!-- Messages -->
    <div class="messages" id="messages">
      <!-- Initial bot message injected by JS -->
    </div>

    <!-- Typing indicator -->
    <div class="message-row bot typing-row" id="typing-row">
      <div class="msg-avatar">🤖</div>
      <div class="typing-bubble">
        <div class="dot"></div>
        <div class="dot"></div>
        <div class="dot"></div>
      </div>
    </div>

    <!-- Quick replies -->
    <div class="quick-replies" id="quick-replies">
      <button class="qr-btn" data-msg="What can you help me with?">📋 What can you do?</button>
      <button class="qr-btn" data-msg="Give me practice questions on algebra">✏️ Practice questions</button>
      <button class="qr-btn" data-msg="Explain a topic step by step">🔢 Step by step</button>
    </div>

    <!-- Input -->
    <div class="input-area">
      <textarea
        id="msg-input"
        placeholder="Ask me anything about maths..."
        rows="1"
        autocomplete="off"
      ></textarea>
      <button id="send-btn" title="Send">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
          <line x1="22" y1="2" x2="11" y2="13"></line>
          <polygon points="22 2 15 22 11 13 2 9 22 2"></polygon>
        </svg>
      </button>
    </div>

    <!-- Footer -->
    <div class="chat-footer">Study Buddy Bot · Mathematics Tutor · Not for homework completion</div>
  </div>

  <script>
    const messagesEl = document.getElementById('messages');
    const inputEl    = document.getElementById('msg-input');
    const sendBtn    = document.getElementById('send-btn');
    const typingRow  = document.getElementById('typing-row');
    const qrArea     = document.getElementById('quick-replies');

    // Generate a session ID (persisted per tab)
    let sessionId = sessionStorage.getItem('sb_session');
    if (!sessionId) {
      sessionId = 'web_' + Math.random().toString(36).slice(2);
      sessionStorage.setItem('sb_session', sessionId);
    }

    // ── Render helpers ────────────────────────────────────────────────────────

    function formatBotText(text) {
      // Convert *bold* and markdown-ish formatting to HTML
      return text
        .replace(/\*([^*]+)\*/g, '<strong>$1</strong>')
        .replace(/_(.*?)_/g, '<em>$1</em>')
        .replace(/\n/g, '<br>');
    }

    function now() {
      return new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    }

    function appendMessage(text, sender, escalated) {
      const row = document.createElement('div');
      row.className = `message-row ${sender}`;

      const avatar = document.createElement('div');
      avatar.className = 'msg-avatar';
      avatar.textContent = sender === 'bot' ? '🤖' : '🧑';

      const bubble = document.createElement('div');
      bubble.className = 'bubble' + (escalated ? ' escalation' : '');

      if (sender === 'bot') {
        bubble.innerHTML = formatBotText(text);
      } else {
        bubble.textContent = text;
      }

      const ts = document.createElement('div');
      ts.className = 'timestamp';
      ts.textContent = now();

      const wrapper = document.createElement('div');
      if (sender === 'bot') {
        wrapper.appendChild(avatar);
        const inner = document.createElement('div');
        inner.appendChild(bubble);
        inner.appendChild(ts);
        wrapper.appendChild(inner);
      } else {
        const inner = document.createElement('div');
        inner.appendChild(bubble);
        inner.appendChild(ts);
        wrapper.appendChild(inner);
        wrapper.appendChild(avatar);
      }
      wrapper.className = `message-row ${sender}`;
      messagesEl.appendChild(wrapper);
      scrollToBottom();
    }

    function scrollToBottom() {
      messagesEl.scrollTop = messagesEl.scrollHeight;
    }

    function showTyping() {
      typingRow.classList.add('active');
      messagesEl.appendChild(typingRow);
      scrollToBottom();
    }

    function hideTyping() {
      typingRow.classList.remove('active');
    }

    // ── Send message ──────────────────────────────────────────────────────────

    async function sendMessage(text) {
      if (!text.trim()) return;
      appendMessage(text, 'user', false);
      inputEl.value = '';
      inputEl.style.height = 'auto';
      sendBtn.disabled = true;
      qrArea.style.display = 'none';

      showTyping();

      try {
        const res = await fetch('/chat', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ sessionId, message: text }),
        });
        const data = await res.json();
        hideTyping();
        // Simulate slight delay like a real bot
        setTimeout(() => {
          appendMessage(data.reply, 'bot', data.escalated);
          sendBtn.disabled = false;
          qrArea.style.display = 'flex';
        }, 200);
      } catch (err) {
        hideTyping();
        appendMessage('⚠️ Oops! Something went wrong. Please try again.', 'bot', false);
        sendBtn.disabled = false;
        qrArea.style.display = 'flex';
      }
    }

    // ── Event listeners ───────────────────────────────────────────────────────

    sendBtn.addEventListener('click', () => sendMessage(inputEl.value));

    inputEl.addEventListener('keydown', (e) => {
      if (e.key === 'Enter' && !e.shiftKey) {
        e.preventDefault();
        sendMessage(inputEl.value);
      }
    });

    // Auto-resize textarea
    inputEl.addEventListener('input', () => {
      inputEl.style.height = 'auto';
      inputEl.style.height = Math.min(inputEl.scrollHeight, 100) + 'px';
    });

    // Topic pills
    document.querySelectorAll('.topic-pill').forEach(btn => {
      btn.addEventListener('click', () => sendMessage(btn.dataset.msg));
    });

    // Quick reply buttons
    qrArea.querySelectorAll('.qr-btn').forEach(btn => {
      btn.addEventListener('click', () => sendMessage(btn.dataset.msg));
    });

    // ── Initial greeting ──────────────────────────────────────────────────────
    window.addEventListener('DOMContentLoaded', () => {
      fetch('/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ sessionId, message: 'hello' }),
      })
        .then(r => r.json())
        .then(d => appendMessage(d.reply, 'bot', false))
        .catch(() => appendMessage('👋 Hi! I\'m Study Buddy Bot — ask me anything about maths!', 'bot', false));
    });
  </script>
</body>
</html>
