<!doctype html>
<html lang="de">
<head>
<meta charset="utf-8">
<title>Mein AFK-Dashboard (Demo)</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
  body { font-family: system-ui, sans-serif; max-width: 700px; margin: 40px auto; padding: 0 20px; background: #f5f5f7; color: #222; }
  h1 { color: #2a6cbf; }
  .karte { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin-bottom: 20px; }
  .demo { background: #fff4d6; border-left: 4px solid #e0a800; }
  input { width: 100%; box-sizing: border-box; padding: 8px; margin: 6px 0; border: 1px solid #ccc; border-radius: 6px; font-size: 1rem; }
  button { background: #2a6cbf; color: white; border: none; padding: 10px 16px; border-radius: 6px; font-size: 1rem; cursor: pointer; margin: 4px 4px 0 0; }
  button:hover { background: #1e4f8f; }
  button.rot { background: #a33; } button.rot:hover { background: #811; }
  button.grau { background: #777; } button.grau:hover { background: #555; }
  .log { background: #111; color: #8f8; font-family: monospace; font-size: 0.8rem; max-height: 140px; overflow-y: auto; padding: 10px; border-radius: 6px; white-space: pre-wrap; margin-top: 10px; }
  .code { background: #fff4d6; border-left: 4px solid #e0a800; padding: 12px; margin: 10px 0; border-radius: 6px; }
  .code b { font-size: 1.4rem; letter-spacing: 3px; }
  .status { font-weight: bold; }
  .online { color: #2a9d2a; } .offline { color: #a33; } .wartet { color: #c77d00; }
  .klein { color: #666; font-size: 0.9rem; }
</style>
</head>
<body>
  <h1>Mein AFK-Dashboard</h1>

  <div class="karte demo">
    <strong>Demo-Modus:</strong> Du kannst Konten hinzufügen (sie bleiben in deinem
    Browser gespeichert) und Start/Stopp ausprobieren, aber es wird <b>nichts</b>
    mit Minecraft verbunden und kein echter Microsoft-Login gemacht.
  </div>

  <div class="karte">
    <h2>Minecraft-Konto hinzufügen</h2>
    <input id="n-label" placeholder="Name für dich, z.B. Mein Zweitaccount">
    <input id="n-host" placeholder="Server-Adresse, z.B. play.deinserver.de">
    <input id="n-port" placeholder="Port (Standard 25565)">
    <button onclick="hinzufuegen()">Hinzufügen</button>
  </div>

  <div id="liste"></div>

<script>
  const $ = id => document.getElementById(id);
  const esc = s => String(s ?? '').replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
  const TEXT = {offline:'offline', login:'wartet auf Microsoft-Login (Demo)', online:'online (AFK, Demo)'};
  const KLASSE = {offline:'offline', login:'wartet', online:'online'};

  let konten = [];
  try { konten = JSON.parse(localStorage.getItem('afk-konten') || '[]'); } catch (_) {}
  konten.forEach(k => { k.status = 'offline'; k.log = k.log || []; });

  const timer = {};
  function speichern() {
    try { localStorage.setItem('afk-konten', JSON.stringify(konten.map(({id,label,host,port}) => ({id,label,host,port})))); } catch (_) {}
  }
  function log(k, text) {
    k.log.push('[' + new Date().toLocaleTimeString('de-DE') + '] ' + text);
    if (k.log.length > 50) k.log.shift();
  }

  function hinzufuegen() {
    const label = $('n-label').value.trim(), host = $('n-host').value.trim();
    if (!label || !host) { alert('Bitte Name und Server-Adresse eintragen.'); return; }
    konten.push({ id: Date.now(), label, host, port: parseInt($('n-port').value) || 25565, status: 'offline', log: [], sek: 0 });
    $('n-label').value = $('n-host').value = $('n-port').value = '';
    speichern(); zeichnen();
  }

  function starten(id) {
    const k = konten.find(x => x.id === id);
    if (!k || k.status !== 'offline') return;
    k.status = 'login';
    log(k, 'Warte auf Microsoft-Login (Simulation) ...');
    zeichnen();
  }
  function loginBestaetigen(id) {
    const k = konten.find(x => x.id === id);
    k.status = 'online'; k.sek = 0;
    log(k, 'Eingeloggt auf ' + k.host + ':' + k.port + ' (Simulation). AFK-Routine läuft.');
    timer[id] = setInterval(() => { k.sek++; if (k.sek % 20 === 0) { log(k, 'Bot schaut sich um und springt. (Simulation)'); zeichnen(); } }, 1000);
    zeichnen();
  }
  function stoppen(id) {
    const k = konten.find(x => x.id === id);
    clearInterval(timer[id]);
    if (k.status !== 'offline') log(k, 'Bot gestoppt.');
    k.status = 'offline';
    zeichnen();
  }
  function entfernen(id) {
    if (!confirm('Dieses Konto entfernen?')) return;
    clearInterval(timer[id]);
    konten = konten.filter(x => x.id !== id);
    speichern(); zeichnen();
  }

  function zeichnen() {
    $('liste').innerHTML = konten.length ? konten.map(k => `
      <div class="karte">
        <h2>${esc(k.label)}</h2>
        <p class="klein">Server: ${esc(k.host)}:${esc(k.port)}</p>
        <p>Status: <span class="status ${KLASSE[k.status]}">${TEXT[k.status]}</span></p>
        ${k.status === 'login' ? `<div class="code"><b>DEMO</b><br>Hier würde bei der echten Version ein Microsoft-Link mit Code stehen.<br>
          <button onclick="loginBestaetigen(${k.id})">Demo-Login bestätigen</button></div>` : ''}
        <button onclick="starten(${k.id})">Starten</button>
        <button class="rot" onclick="stoppen(${k.id})">Stoppen</button>
        <button class="grau" onclick="entfernen(${k.id})">Entfernen</button>
        <div class="log">${esc(k.log.join('\n')) || 'Noch keine Einträge.'}</div>
      </div>`).join('') : '<div class="karte"><p>Noch kein Konto hinzugefügt.</p></div>';
  }
  zeichnen();
</script>
</body>
</html>
