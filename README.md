<!DOCTYPE html>
<html lang="en" class="dark">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <meta name="theme-color" content="#060b16" />
  <meta name="apple-mobile-web-app-capable" content="yes" />
  <title>Jarvis — Personal AI Assistant</title>
  <script src="https://cdn.tailwindcss.com/3.4.17"></script>
  <script src="https://unpkg.com/lucide@0.460.0/dist/umd/lucide.min.js"></script>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet" />
  <style>
    :root {
      --bg-deep: #060b16;
      --bg-panel: #0c1426;
      --bg-card: #111c33;
      --accent: #38bdf8;
      --accent-glow: rgba(56, 189, 248, 0.35);
    }
    * { box-sizing: border-box; }
    body {
      font-family: 'Inter', system-ui, -apple-system, sans-serif;
      background-color: var(--bg-deep);
      color: #e2e8f0;
      margin: 0;
      min-height: 100vh;
      overflow-x: hidden;
    }
    .mono { font-family: 'JetBrains Mono', monospace; }
    .glass {
      background: rgba(12, 20, 38, 0.82);
      backdrop-filter: blur(12px);
      border: 1px solid rgba(56, 189, 248, 0.16);
    }
    .orb-ring {
      animation: spinSlow 18s linear infinite;
      transform-origin: center;
    }
    .orb-ring-rev {
      animation: spinSlowRev 12s linear infinite;
      transform-origin: center;
    }
    .orb-pulse {
      animation: pulseCore 2.8s ease-in-out infinite;
    }
    .orb-listening .orb-pulse {
      animation: pulseListen 0.9s ease-in-out infinite;
    }
    .orb-thinking .orb-ring {
      animation: spinSlow 2.2s linear infinite;
    }
    @keyframes spinSlow {
      from { transform: rotate(0deg); }
      to { transform: rotate(360deg); }
    }
    @keyframes spinSlowRev {
      from { transform: rotate(360deg); }
      to { transform: rotate(0deg); }
    }
    @keyframes pulseCore {
      0%, 100% { transform: scale(1); opacity: 0.85; }
      50% { transform: scale(1.07); opacity: 1; }
    }
    @keyframes pulseListen {
      0%, 100% { transform: scale(0.96); opacity: 0.75; }
      50% { transform: scale(1.14); opacity: 1; }
    }
    /* Custom scrollbar */
    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-track { background: transparent; }
    ::-webkit-scrollbar-thumb { background: rgba(56, 189, 248, 0.25); border-radius: 9999px; }
    ::-webkit-scrollbar-thumb:hover { background: rgba(56, 189, 248, 0.45); }
  </style>
</head>
<body class="flex flex-col h-screen max-h-screen overflow-hidden bg-[#060b16] text-slate-200">

  <!-- Top Header -->
  <header class="glass shrink-0 z-20 px-4 py-2.5 flex items-center justify-between gap-2 border-b border-sky-500/20">
    <div class="flex items-center gap-2.5 min-w-0">
      <div class="w-8 h-8 rounded-full bg-sky-500/15 border border-sky-400/40 flex items-center justify-center shrink-0">
        <span class="w-2.5 h-2.5 rounded-full bg-sky-400 shadow-[0_0_10px_#38bdf8]"></span>
      </div>
      <div class="min-w-0">
        <div class="flex items-center gap-2">
          <h1 class="text-sm sm:text-base font-bold tracking-wider text-sky-300 uppercase">J.A.R.V.I.S.</h1>
          <span id="status-badge" class="hidden sm:inline-flex items-center gap-1 text-[10px] mono px-2 py-0.5 rounded-full bg-emerald-500/15 text-emerald-300 border border-emerald-500/30">
            <span class="w-1.5 h-1.5 rounded-full bg-emerald-400"></span> ONLINE
          </span>
        </div>
        <p id="live-clock" class="text-[11px] text-slate-400 mono truncate">Initializing...</p>
      </div>
    </div>

    <div class="flex items-center gap-1.5 sm:gap-2 shrink-0">
      <div id="weather-pill" class="hidden md:flex items-center gap-1.5 px-2.5 py-1 rounded-full bg-slate-800/70 border border-slate-700/80 text-xs text-slate-300">
        <i data-lucide="cloud-sun" class="w-3.5 h-3.5 text-amber-300"></i>
        <span id="weather-text">Kanigiri: --°C</span>
      </div>
      <button id="tts-toggle" type="button" title="Toggle Voice Response" class="flex items-center gap-1.5 px-2.5 py-1.5 rounded-lg bg-slate-800/80 hover:bg-slate-700/80 border border-slate-700 text-xs font-medium transition">
        <i id="tts-icon" data-lucide="volume-2" class="w-4 h-4 text-sky-400"></i>
        <span id="tts-label" class="hidden sm:inline">Voice On</span>
      </button>
      <button id="open-settings-btn" type="button" title="Assistant Settings" class="p-1.5 sm:px-2.5 sm:py-1.5 rounded-lg bg-slate-800/80 hover:bg-slate-700/80 border border-slate-700 text-xs font-medium flex items-center gap-1.5 transition">
        <i data-lucide="sliders-horizontal" class="w-4 h-4 text-slate-300"></i>
        <span class="hidden sm:inline">Settings</span>
      </button>
    </div>
  </header>

  <!-- Main Workspace -->
  <div class="flex-1 flex flex-col md:flex-row min-h-0 overflow-hidden">

    <!-- Desktop Sidebar Navigation -->
    <nav class="hidden md:flex flex-col w-56 shrink-0 glass border-r border-sky-500/15 p-3 gap-1.5">
      <button data-tab="console" class="tab-btn flex items-center gap-3 px-3 py-2.5 rounded-xl text-sm font-medium bg-sky-500/15 text-sky-300 border border-sky-500/30 transition text-left">
        <i data-lucide="terminal" class="w-4 h-4"></i>
        <span>Command Console</span>
      </button>
      <button data-tab="tasks" class="tab-btn flex items-center gap-3 px-3 py-2.5 rounded-xl text-sm font-medium text-slate-400 hover:text-slate-200 hover:bg-slate-800/60 border border-transparent transition text-left">
        <i data-lucide="check-square" class="w-4 h-4"></i>
        <span>Tasks &amp; Timers</span>
        <span id="nav-task-count" class="ml-auto text-[11px] mono px-1.5 py-0.5 rounded bg-slate-800 text-slate-300">0</span>
      </button>
      <button data-tab="notes" class="tab-btn flex items-center gap-3 px-3 py-2.5 rounded-xl text-sm font-medium text-slate-400 hover:text-slate-200 hover:bg-slate-800/60 border border-transparent transition text-left">
        <i data-lucide="notebook-pen" class="w-4 h-4"></i>
        <span>Notes &amp; Briefing</span>
        <span id="nav-note-count" class="ml-auto text-[11px] mono px-1.5 py-0.5 rounded bg-slate-800 text-slate-300">0</span>
      </button>

      <div class="mt-auto pt-3 border-t border-slate-800/80 space-y-2">
        <div class="p-3 rounded-xl bg-slate-900/80 border border-slate-800">
          <p class="text-[11px] uppercase tracking-wider text-slate-400 font-semibold mb-1">Active Profile</p>
          <p id="sidebar-user-name" class="text-xs font-medium text-slate-200 truncate">Srinivas</p>
          <p id="sidebar-engine-mode" class="text-[11px] text-sky-400 mono mt-0.5">Local Smart Engine</p>
        </div>
      </div>
    </nav>

    <!-- Tab Content Area -->
    <main class="flex-1 flex flex-col min-h-0 overflow-hidden relative">

      <!-- TAB 1: CONSOLE -->
      <section id="panel-console" class="tab-panel flex-1 flex flex-col min-h-0 overflow-hidden">
        <!-- Jarvis Orb & Status Strip -->
        <div class="shrink-0 px-4 py-3 border-b border-slate-800/70 bg-[#080f1e]/90 flex flex-col sm:flex-row items-center gap-3 sm:gap-5">
          <div id="jarvis-orb" class="relative w-16 h-16 sm:w-20 sm:h-20 shrink-0 flex items-center justify-center">
            <svg viewBox="0 0 100 100" class="w-full h-full">
              <defs>
                <radialGradient id="coreGrad" cx="50%" cy="50%" r="50%">
                  <stop offset="0%" stop-color="#7dd3fc" stop-opacity="0.95" />
                  <stop offset="60%" stop-color="#0284c7" stop-opacity="0.45" />
                  <stop offset="100%" stop-color="#060b16" stop-opacity="0" />
                </radialGradient>
              </defs>
              <circle cx="50" cy="50" r="45" fill="none" stroke="rgba(56,189,248,0.22)" stroke-width="1.5" stroke-dasharray="10 6" class="orb-ring" />
              <circle cx="50" cy="50" r="35" fill="none" stroke="rgba(14,165,233,0.45)" stroke-width="2" stroke-dasharray="18 10 4 10" class="orb-ring-rev" />
              <circle cx="50" cy="50" r="24" fill="url(#coreGrad)" class="orb-pulse" />
              <circle cx="50" cy="50" r="10" fill="#bae6fd" class="orb-pulse" />
            </svg>
          </div>
          <div class="flex-1 text-center sm:text-left min-w-0">
            <p id="jarvis-greeting" class="text-sm sm:text-base font-semibold text-slate-100">Good morning, Srinivas. Systems are ready.</p>
            <p id="jarvis-subtitle" class="text-xs text-slate-400 mt-0.5">Ask me to manage tasks, capture notes, start timers, run calculations, or give a daily briefing.</p>
          </div>
          <button id="clear-chat-btn" type="button" class="shrink-0 px-2.5 py-1.5 rounded-lg text-xs text-slate-400 hover:text-slate-200 hover:bg-slate-800 border border-slate-800 transition flex items-center gap-1">
            <i data-lucide="rotate-ccw" class="w-3.5 h-3.5"></i>
            <span>Reset Chat</span>
          </button>
        </div>

        <!-- Chat Feed -->
        <div id="chat-feed" class="flex-1 overflow-y-auto px-4 py-4 space-y-3"></div>

        <!-- Quick Command Chips -->
        <div class="shrink-0 px-4 py-2 border-t border-slate-800/60 bg-[#080f1e]/80 flex items-center gap-2 overflow-x-auto">
          <button type="button" data-cmd="Give me my daily briefing" class="quick-cmd shrink-0 px-3 py-1 rounded-full text-xs bg-sky-500/10 hover:bg-sky-500/20 text-sky-300 border border-sky-500/25 transition">Daily Briefing</button>
          <button type="button" data-cmd="Add task Check quarry spares inventory" class="quick-cmd shrink-0 px-3 py-1 rounded-full text-xs bg-slate-800 hover:bg-slate-700 text-slate-300 border border-slate-700 transition">+ Quick Task</button>
          <button type="button" data-cmd="Start a 5 minute timer" class="quick-cmd shrink-0 px-3 py-1 rounded-full text-xs bg-slate-800 hover:bg-slate-700 text-slate-300 border border-slate-700 transition">5m Timer</button>
          <button type="button" data-cmd="Calculate 45000 * 1.18" class="quick-cmd shrink-0 px-3 py-1 rounded-full text-xs bg-slate-800 hover:bg-slate-700 text-slate-300 border border-slate-700 transition">GST Calc</button>
          <button type="button" data-cmd="Say hello in Telugu" class="quick-cmd shrink-0 px-3 py-1 rounded-full text-xs bg-slate-800 hover:bg-slate-700 text-slate-300 border border-slate-700 transition">Telugu Greeting</button>
          <button type="button" data-cmd="Help" class="quick-cmd shrink-0 px-3 py-1 rounded-full text-xs bg-slate-800 hover:bg-slate-700 text-slate-300 border border-slate-700 transition">Commands</button>
        </div>

        <!-- Input Bar -->
        <form id="chat-form" class="shrink-0 p-3 sm:p-4 glass border-t border-sky-500/20 flex items-center gap-2">
          <button id="mic-btn" type="button" title="Voice Input" class="p-2.5 rounded-xl bg-slate-800 hover:bg-slate-700 border border-slate-700 text-sky-400 transition shrink-0">
            <i data-lucide="mic" class="w-5 h-5"></i>
          </button>
          <input id="chat-input" type="text" autocomplete="off" placeholder="Message Jarvis or type a command (e.g., 'add task...', 'calculate 25*18')..." class="flex-1 min-w-0 bg-slate-900/90 border border-slate-700 focus:border-sky-400 focus:outline-none rounded-xl px-3.5 py-2.5 text-sm text-slate-100 placeholder-slate-500" />
          <button type="submit" class="px-4 py-2.5 rounded-xl bg-sky-500 hover:bg-sky-400 text-slate-950 font-semibold text-sm flex items-center gap-1.5 transition shrink-0">
            <span>Send</span>
            <i data-lucide="send" class="w-4 h-4"></i>
          </button>
        </form>
      </section>

      <!-- TAB 2: TASKS & TIMERS -->
      <section id="panel-tasks" class="tab-panel hidden flex-1 overflow-y-auto p-4 sm:p-6 space-y-6">
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 max-w-5xl mx-auto">
          <!-- Tasks Card -->
          <div class="glass rounded-2xl p-4 sm:p-5 space-y-4">
            <div class="flex items-center justify-between">
              <div>
                <h2 class="text-base font-semibold text-slate-100">Action Queue</h2>
                <p class="text-xs text-slate-400">Saved automatically on this device</p>
              </div>
              <span id="tasks-summary-pill" class="text-xs mono px-2.5 py-1 rounded-full bg-sky-500/15 text-sky-300 border border-sky-500/30">0 pending</span>
            </div>

            <form id="task-form" class="flex gap-2">
              <input id="task-input" type="text" placeholder="Add a new task..." class="flex-1 min-w-0 bg-slate-900 border border-slate-700 focus:border-sky-400 focus:outline-none rounded-xl px-3 py-2 text-sm text-slate-100" />
              <select id="task-priority" class="bg-slate-900 border border-slate-700 rounded-xl px-2.5 py-2 text-xs text-slate-200 focus:outline-none">
                <option value="normal">Normal</option>
                <option value="high">High</option>
              </select>
              <button type="submit" class="px-3.5 py-2 rounded-xl bg-sky-500 hover:bg-sky-400 text-slate-950 font-semibold text-sm transition shrink-0">Add</button>
            </form>

            <div id="task-list" class="space-y-2 max-h-80 overflow-y-auto pr-1"></div>
          </div>

          <!-- Timer Card -->
          <div class="glass rounded-2xl p-4 sm:p-5 flex flex-col justify-between space-y-5">
            <div>
              <h2 class="text-base font-semibold text-slate-100">Focus &amp; Countdown Timer</h2>
              <p class="text-xs text-slate-400">Audio chime alerts you when time is up</p>
            </div>

            <div class="py-6 flex flex-col items-center justify-center rounded-2xl bg-slate-900/80 border border-slate-800">
              <div id="timer-display" class="text-4xl sm:text-5xl font-bold mono text-sky-300 tracking-wider">25:00</div>
              <p id="timer-status" class="text-xs text-slate-400 mt-2">Ready</p>
            </div>

            <div class="grid grid-cols-3 gap-2">
              <button type="button" data-mins="5" class="timer-preset py-2 rounded-xl bg-slate-800 hover:bg-slate-700 text-xs font-medium border border-slate-700 transition">5 min</button>
              <button type="button" data-mins="15" class="timer-preset py-2 rounded-xl bg-slate-800 hover:bg-slate-700 text-xs font-medium border border-slate-700 transition">15 min</button>
              <button type="button" data-mins="25" class="timer-preset py-2 rounded-xl bg-slate-800 hover:bg-slate-700 text-xs font-medium border border-slate-700 transition">25 min</button>
            </div>

            <div class="flex items-center gap-2">
              <button id="timer-start-btn" type="button" class="flex-1 py-2.5 rounded-xl bg-sky-500 hover:bg-sky-400 text-slate-950 font-semibold text-sm transition">Start</button>
              <button id="timer-pause-btn" type="button" class="flex-1 py-2.5 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-200 font-medium text-sm border border-slate-700 transition">Pause</button>
              <button id="timer-reset-btn" type="button" class="px-4 py-2.5 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-300 font-medium text-sm border border-slate-700 transition">Reset</button>
            </div>
          </div>
        </div>
      </section>

      <!-- TAB 3: NOTES & BRIEFING -->
      <section id="panel-notes" class="tab-panel hidden flex-1 overflow-y-auto p-4 sm:p-6 space-y-6">
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 max-w-5xl mx-auto">
          <!-- Notes Card -->
          <div class="glass rounded-2xl p-4 sm:p-5 space-y-4">
            <div class="flex items-center justify-between">
              <div>
                <h2 class="text-base font-semibold text-slate-100">Quick Notes</h2>
                <p class="text-xs text-slate-400">Capture ideas, parts lists, or reminders</p>
              </div>
            </div>

            <form id="note-form" class="space-y-2">
              <textarea id="note-input" rows="2" placeholder="Write a quick note..." class="w-full bg-slate-900 border border-slate-700 focus:border-sky-400 focus:outline-none rounded-xl p-3 text-sm text-slate-100 resize-none"></textarea>
              <button type="submit" class="w-full py-2 rounded-xl bg-sky-500 hover:bg-sky-400 text-slate-950 font-semibold text-sm transition">Save Note</button>
            </form>

            <div id="note-list" class="space-y-2 max-h-72 overflow-y-auto pr-1"></div>
          </div>

          <!-- Daily Briefing Card -->
          <div class="glass rounded-2xl p-4 sm:p-5 space-y-4">
            <div class="flex items-center justify-between">
              <div>
                <h2 class="text-base font-semibold text-slate-100">Daily Briefing</h2>
                <p class="text-xs text-slate-400">Live snapshot for your day</p>
              </div>
              <button id="refresh-briefing-btn" type="button" class="px-2.5 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 border border-slate-700 text-xs text-sky-300 transition flex items-center gap-1">
                <i data-lucide="refresh-cw" class="w-3.5 h-3.5"></i>
                <span>Refresh</span>
              </button>
            </div>
            <div id="briefing-card" class="space-y-3 text-sm text-slate-300"></div>
          </div>
        </div>
      </section>

    </main>
  </div>

  <!-- Mobile Bottom Navigation -->
  <nav class="md:hidden shrink-0 glass border-t border-sky-500/20 flex items-center justify-around py-1.5 px-2 z-20">
    <button data-tab="console" class="tab-btn flex flex-col items-center gap-0.5 px-3 py-1 rounded-lg text-sky-300 text-[11px] font-medium">
      <i data-lucide="terminal" class="w-4 h-4"></i>
      <span>Console</span>
    </button>
    <button data-tab="tasks" class="tab-btn flex flex-col items-center gap-0.5 px-3 py-1 rounded-lg text-slate-400 text-[11px] font-medium">
      <i data-lucide="check-square" class="w-4 h-4"></i>
      <span>Tasks</span>
    </button>
    <button data-tab="notes" class="tab-btn flex flex-col items-center gap-0.5 px-3 py-1 rounded-lg text-slate-400 text-[11px] font-medium">
      <i data-lucide="notebook-pen" class="w-4 h-4"></i>
      <span>Notes</span>
    </button>
  </nav>

  <!-- Settings Modal -->
  <div id="settings-modal" class="hidden fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-sm flex items-center justify-center p-4">
    <div class="glass rounded-2xl w-full max-w-md p-5 space-y-4 border border-sky-500/30 shadow-2xl">
      <div class="flex items-center justify-between">
        <h3 class="text-base font-semibold text-slate-100">Jarvis Settings</h3>
        <button id="close-settings-btn" type="button" class="p-1 rounded-lg hover:bg-slate-800 text-slate-400 hover:text-slate-200">
          <i data-lucide="x" class="w-5 h-5"></i>
        </button>
      </div>
      <div class="space-y-3">
        <div>
          <label for="setting-username" class="block text-xs font-medium text-slate-300 mb-1">Your Name</label>
          <input id="setting-username" type="text" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-sm text-slate-100 focus:border-sky-400 focus:outline-none" />
        </div>
        <div>
          <label for="setting-apikey" class="block text-xs font-medium text-slate-300 mb-1">Optional Gemini API Key (for Cloud AI mode)</label>
          <input id="setting-apikey" type="password" placeholder="Leave blank to use built-in Local Smart Engine" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-sm text-slate-100 focus:border-sky-400 focus:outline-none" />
          <p class="text-[11px] text-slate-400 mt-1">Stored only in your 
