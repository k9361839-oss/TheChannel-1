<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>אלעדים על המפה - המערכת הסופית</title>
    <script src="https://www.gstatic.com/firebasejs/9.15.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.15.0/firebase-database-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.15.0/firebase-auth-compat.js"></script>
    <style>
        :root { --main-green: #008069; --bg-chat: #efe7de; --msg-me: #dcf8c6; --msg-other: #ffffff; }
        body, html { margin: 0; padding: 0; height: 100%; font-family: 'Segoe UI', sans-serif; overflow: hidden; }
        #main-wrapper { width: 100%; height: 100%; display: flex; flex-direction: column; background: var(--bg-chat); position: relative; }
        #thread-panel { position: fixed; left: 0; top: 0; bottom: 0; width: 340px; background: white; z-index: 5000; display: none; flex-direction: column; box-shadow: 2px 0 20px rgba(0,0,0,0.4); border-right: 1px solid #ddd; }
        header { background: white; padding: 10px 20px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #ddd; height: 60px; box-sizing: border-box; }
        #chat-content { flex: 1; overflow-y: auto; padding: 15px; display: flex; flex-direction: column; gap: 10px; }
        .message { max-width: 85%; padding: 10px; border-radius: 8px; box-shadow: 0 1px 2px rgba(0,0,0,0.1); cursor: pointer; position: relative; }
        .message.me { align-self: flex-end; background: var(--msg-me); }
        .message.other { align-self: flex-start; background: var(--msg-other); }
        .tools { display: flex; gap: 12px; margin-top: 8px; font-size: 0.8rem; border-top: 1px solid rgba(0,0,0,0.05); padding-top: 5px; opacity: 0.6; }
        .tools span:hover { opacity: 1; color: var(--main-green); cursor: pointer; }
        .react-line { display: flex; gap: 4px; margin-top: 5px; }
        .react-chip { background: white; border: 1px solid #eee; border-radius: 12px; padding: 1px 6px; font-size: 0.7rem; font-weight: bold; }
        footer { padding: 10px; background: #f0f2f5; display: flex; gap: 10px; align-items: center; border-top: 1px solid #ddd; }
        .in-box { flex: 1; background: white; border-radius: 25px; padding: 5px 15px; display: flex; }
        input { border: none; flex: 1; padding: 8px; outline: none; background: transparent; font-size: 1rem; }
        .send-btn { background: var(--main-green); color: white; width: 42px; height: 42px; border-radius: 50%; border: none; cursor: pointer; font-size: 1.3rem; display: flex; align-items: center; justify-content: center; }
        #t-head { padding: 15px; background: #075e54; color: white; display: flex; justify-content: space-between; align-items: center; }
        #t-body { flex: 1; overflow-y: auto; padding: 15px; background: var(--bg-chat); display: flex; flex-direction: column; gap: 10px; }
    </style>
</head>
<body>
<div id="main-wrapper">
    <header>
        <div style="display:flex; align-items:center; gap:10px;">
            <img src="https://k9361839-oss.github.io/The-Channel/אלעדים על המפה.jpg" style="width:40px; height:40px; border-radius:50%; object-fit:cover;">
            <b>אלעדים על המפה</b>
        </div>
        <div id="auth-box"><button onclick="login()" style="cursor:pointer; border-radius:20px; padding:5px 15px;">כניסה 🔑</button></div>
    </header>
    <div id="chat-content"></div>
    <footer>
        <button style="background:none; border:none; font-size:1.6rem; cursor:pointer;" onclick="addEmoji('main-in')">😊</button>
        <div class="in-box"><input type="text" id="main-in" placeholder="הודעה..." onkeypress="if(event.key==='Enter') sendMsg('main-in', 'messages')"></div>
        <button class="send-btn" onclick="sendMsg('main-in', 'messages')">➤</button>
    </footer>
</div>
<div id="thread-panel">
    <div id="t-head"><b>שרשור תגובות</b><button onclick="closeThread()" style="background:none; border:none; color:white; font-size:1.5rem; cursor:pointer;">×</button></div>
    <div id="t-body"></div>
    <footer style="flex-direction: column; height: auto; padding: 10px; background: #eee;">
        <div class="in-box" style="width:100%; box-sizing:border-box;"><input type="text" id="t-in" placeholder="הגב בשרשור..." onkeypress="if(event.key==='Enter') reply()"></div>
        <button class="send-btn" style="margin-top:5px;" onclick="reply()">➤</button>
    </footer>
</div>
<script>
    const firebaseConfig = {
        apiKey: "AIzaSyCBOb3dKOpwZ3anv5N8Lqlv2x3PPuokBFg",
        authDomain: "eladim-chat-v2.firebaseapp.com",
        databaseURL: "https://eladim-chat-v2-default-rtdb.europe-west1.firebasedatabase.app",
        projectId: "eladim-chat-v2",
        storageBucket: "eladim-chat-v2.firebasestorage.app",
        messagingSenderId: "514164337891",
        appId: "1:514164337891:web:b0a098a9c063b3a2bb29d6"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database(), auth = firebase.auth();
    let me = null, activeT = null;
    const ADMINS = ["your-email@gmail.com"]; // שנה למייל שלך!
    function login() { auth.signInWithPopup(new firebase.auth.GoogleAuthProvider()); }
    auth.onAuthStateChanged(u => { me = u; if(u) document.getElementById('auth-box').innerHTML = `<img src="${u.photoURL}" style="width:35px; border-radius:50%; border:2px solid var(--main-green);">`; });
    function addEmoji(id) { const e = prompt("אימוג'י:"); if(e) document.getElementById(id).value += e; }
    function sendMsg(id, p) { const v = document.getElementById(id).value.trim(); if(!v || !me) return; db.ref(p).push({ uid: me.uid, name: me.displayName, text: v, time: Date.now() }); document.getElementById(id).value = ""; }
    function reply() { if(activeT) sendMsg('t-in', `messages/${activeT}/replies`); }
    db.ref('messages').on('value', s => { const c = document.getElementById('chat-content'); c.innerHTML = ""; s.forEach(x => render(x, c, 'messages')); c.scrollTop = c.scrollHeight; });
    function render(x, container, path) {
        const d = x.val(), id = x.key;
        const isAdm = me && ADMINS.includes(me.email);
        const canMod = me && (d.uid === me.uid || isAdm);
        const div = document.createElement('div');
        div.className = `message ${me && d.uid === me.uid ? 'me' : 'other'}`;
        div.innerHTML = `<b style="font-size:0.75rem; color:var(--main-green);">${d.name}</b><div onclick="openThread('${id}')">${d.text}</div><div class="react-line" id="r-${id}"></div><div class="tools"><span onclick="react('${id}','👍','${path}')">👍</span> <span onclick="openThread('${id}')">💬 שרשור</span>${canMod ? ` <span onclick="editMsg('${id}','${d.text}','${path}')">✏️</span> <span onclick="delMsg('${id}','${path}')">🗑️</span>` : ''}</div>`;
        container.appendChild(div);
        loadReactions(id, path);
    }
    function openThread(id) { activeT = id; document.getElementById('thread-panel').style.display = 'flex'; db.ref(`messages/${id}/replies`).on('value', s => { const b = document.getElementById('t-body'); b.innerHTML = ""; s.forEach(x => render(x, b, `messages/${id}/replies`)); b.scrollTop = b.scrollHeight; }); }
    function closeThread() { document.getElementById('thread-panel').style.display = 'none'; }
    function react(id, e, p) { if(me) db.ref(`${p}/${id}/reacs/${e}/${me.uid}`).set(true); }
    function loadReactions(id, p) { db.ref(`${p}/${id}/reacs`).on('value', s => { const b = document.getElementById('r-'+id); if(!b) return; b.innerHTML = ""; s.forEach(g => b.innerHTML += `<span class="react-chip">${g.key} ${Object.keys(g.val()).length}</span>`); }); }
    function delMsg(id, p) { if(confirm("למחוק?")) db.ref(`${p}/${id}`).remove(); }
    function editMsg(id, old, p) { const n = prompt("ערוך:", old); if(n) db.ref(`${p}/${id}`).update({text: n}); }
</script>
</body>
</html>
