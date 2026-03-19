# פרויקט צ'אט: אלעדים על המפה - גרסה סופית ויציבה 🚀

זהו הגיבוי המלא של הקוד לאתר הצ'אט. הקוד כולל מערכת שרשורים צפה (שלא שוברת את העיצוב), אימוג'ים, עריכה, מחיקה וניהול מנהלים.

## איך להשתמש בקוד הזה?
יש להעתיק את כל התוכן שמתחת לשורה הזו ולהדביק אותו בקובץ `index.html`.

---

```html
<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>אלעדים על המפה - הגרסה המנצחת</title>
    <script src="[https://www.gstatic.com/firebasejs/9.15.0/firebase-app-compat.js](https://www.gstatic.com/firebasejs/9.15.0/firebase-app-compat.js)"></script>
    <script src="[https://www.gstatic.com/firebasejs/9.15.0/firebase-database-compat.js](https://www.gstatic.com/firebasejs/9.15.0/firebase-database-compat.js)"></script>
    <script src="[https://www.gstatic.com/firebasejs/9.15.0/firebase-auth-compat.js](https://www.gstatic.com/firebasejs/9.15.0/firebase-auth-compat.js)"></script>
    <style>
        :root { --main-green: #008069; --bg-chat: #efe7de; --msg-me: #dcf8c6; --msg-other: #ffffff; }
        body, html { margin: 0; padding: 0; height: 100%; font-family: 'Segoe UI', sans-serif; overflow: hidden; }
        
        /* הצ'אט הראשי תמיד תופס 100% מהמסך */
        #main-container { width: 100%; height: 100%; display: flex; flex-direction: column; background: var(--bg-chat); position: relative; }
        
        /* שרשור צף (Fixed) - פותר את בעיית השטח הלבן */
        #thread-sidebar { 
            position: fixed; left: 0; top: 0; bottom: 0; 
            width: 350px; background: white; z-index: 1000;
            display: none; flex-direction: column;
            box-shadow: 2px 0 15px rgba(0,0,0,0.3);
            border-right: 1px solid #ddd;
        }

        header { background: white; padding: 10px 20px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #ddd; height: 60px; box-sizing: border-box; }
        #chat-flow { flex: 1; overflow-y: auto; padding: 20px; display: flex; flex-direction: column; gap: 10px; }
        
        .msg { max-width: 80%; padding: 10px; border-radius: 8px; box-shadow: 0 1px 1px rgba(0,0,0,0.1); cursor: pointer; position: relative; }
        .msg.me { align-self: flex-end; background: var(--msg-me); }
        .msg.other { align-self: flex-start; background: var(--msg-other); }
        .msg-user { font-size: 0.75rem; font-weight: bold; color: #00a884; display: block; margin-bottom: 3px; }
        
        .msg-tools { display: flex; gap: 12px; margin-top: 8px; font-size: 0.8rem; border-top: 1px solid rgba(0,0,0,0.05); padding-top: 5px; opacity: 0.6; }
        .msg-tools span:hover { opacity: 1; cursor: pointer; color: var(--main-green); }
        
        .reac-tag { background: white; border: 1px solid #ddd; border-radius: 10px; padding: 2px 6px; font-size: 0.75rem; margin-top: 5px; display: inline-block; }

        footer { padding: 10px; background: #f0f2f5; display: flex; gap: 10px; align-items: center; border-top: 1px solid #ddd; }
        .input-wrap { flex: 1; background: white; border-radius: 20px; padding: 5px 15px; display: flex; }
        input { border: none; flex: 1; padding: 8px; outline: none; background: transparent; font-size: 1rem; }
        .btn-circle { background: var(--main-green); color: white; width: 42px; height: 42px; border-radius: 50%; border: none; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 1.3rem; }

        #thread-header { padding: 15px; background: #075e54; color: white; display: flex; justify-content: space-between; align-items: center; }
        #thread-msgs { flex: 1; overflow-y: auto; padding: 15px; background: var(--bg-chat); display: flex; flex-direction: column; gap: 10px; }
    </style>
</head>
<body>

<div id="main-container">
    <header>
        <div style="display:flex; align-items:center; gap:10px;">
            <img src="[https://k9361839-oss.github.io/The-Channel/אלעדים](https://k9361839-oss.github.io/The-Channel/אלעדים) על המפה.jpg" style="width:40px; height:40px; border-radius:50%; object-fit:cover;">
            <b>אלעדים על המפה</b>
        </div>
        <div id="auth-area"><button onclick="login()" style="padding:8px 15px; border-radius:20px; cursor:pointer; background:white; border:1px solid var(--main-green); color:var(--main-green); font-weight:bold;">כניסה 🔑</button></div>
    </header>

    <div id="chat-flow"></div>

    <footer>
        <button style="background:none; border:none; font-size:1.6rem; cursor:pointer;" onclick="addEmoji('main-input')">😊</button>
        <div class="input-wrap"><input type="text" id="main-input" placeholder="כתוב הודעה..." onkeypress="if(event.key==='Enter') sendMessage('main-input', 'messages')"></div>
        <button class="btn-circle" onclick="sendMessage('main-input', 'messages')">➤</button>
    </footer>
</div>

<div id="thread-sidebar">
    <div id="thread-header">
        <b>שרשור תגובות</b>
        <button onclick="closeThread()" style="background:none; border:none; color:white; font-size:1.5rem; cursor:pointer;">×</button>
    </div>
    <div id="thread-msgs"></div>
    <footer style="flex-direction: column; padding: 15px; background: #eee;">
        <div class="input-wrap" style="width: 100%; box-sizing: border-box;"><input type="text" id="thread-input" placeholder="הגב בשרשור..." onkeypress="if(event.key==='Enter') sendReply()"></div>
        <button class="btn-circle" style="margin-top:8px;" onclick="sendReply()">➤</button>
    </footer>
</div>

<script>
    const firebaseConfig = {
        apiKey: "AIzaSyCBOb3dKOpwZ3anv5N8Lqlv2x3PPuokBFg",
        authDomain: "eladim-chat-v2.firebaseapp.com",
        databaseURL: "[https://eladim-chat-v2-default-rtdb.europe-west1.firebasedatabase.app](https://eladim-chat-v2-default-rtdb.europe-west1.firebasedatabase.app)",
        projectId: "eladim-chat-v2",
        storageBucket: "eladim-chat-v2.firebasestorage.app",
        messagingSenderId: "514164337891",
        appId: "1:514164337891:web:b0a098a9c063b3a2bb29d6"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
    const auth = firebase.auth();
    let currentUser = null;
    let activeThreadId = null;

    // --- הגדרת מנהלים (שנה כאן למייל שלך) ---
    const ADMINS = ["your-email@gmail.com"]; 

    function login() { auth.signInWithPopup(new firebase.auth.GoogleAuthProvider()); }

    auth.onAuthStateChanged(user => {
        currentUser = user;
        if(user) document.getElementById('auth-area').innerHTML = `<img src="${user.photoURL}" style="width:35px; border-radius:50%; border:2px solid var(--main-green);">`;
    });

    function addEmoji(id) { 
        const e = prompt("בחר אימוג'י (או הקלד):", "😊 😂 🔥 ❤️"); 
        if(e) document.getElementById(id).value += e; 
    }

    function sendMessage(inputId, path) {
        const text = document.getElementById(inputId).value.trim();
        if(!text || !currentUser) return;
        db.ref(path).push({ uid: currentUser.uid, name: currentUser.displayName, text: text, timestamp: Date.now() });
        document.getElementById(inputId).value = "";
    }

    function sendReply() { if(activeThreadId) sendMessage('thread-input', `messages/${activeThreadId}/replies`); }

    db.ref('messages').on('value', snap => {
        const flow = document.getElementById('chat-flow');
        flow.innerHTML = "";
        snap.forEach(child => renderMessage(child, flow, 'messages'));
        flow.scrollTop = flow.scrollHeight;
    });

    function renderMessage(child, container, basePath) {
        const m = child.val();
        const id = child.key;
        const isAdmin = currentUser && ADMINS.includes(currentUser.email);
        const isOwner = currentUser && (m.uid === currentUser.uid || isAdmin);

        const div = document.createElement('div');
        div.className = `msg ${currentUser && m.uid === currentUser.uid ? 'me' : 'other'}`;
        div.innerHTML = `
            <span class="msg-user">${m.name}</span>
            <div onclick="openThread('${id}')">${m.text}</div>
            <div id="reac-${id}"></div>
            <div class="msg-tools">
                <span onclick="react('${id}', '👍', '${basePath}')">👍</span>
                <span onclick="openThread('${id}')">💬 שרשור</span>
                ${isOwner ? `<span onclick="editMsg('${id}', '${m.text}', '${basePath}')">✏️</span> <span onclick="deleteMsg('${id}', '${basePath}')">🗑️</span>` : ''}
            </div>
        `;
        container.appendChild(div);
        loadReactions(id, basePath);
    }

    function openThread(id) {
        activeThreadId = id;
        document.getElementById('thread-sidebar').style.display = 'flex';
        db.ref(`messages/${id}/replies`).on('value', snap => {
            const tArea = document.getElementById('thread-msgs');
            tArea.innerHTML = "";
            snap.forEach(c => renderMessage(c, tArea, `messages/${id}/replies`));
            tArea.scrollTop = tArea.scrollHeight;
        });
    }

    function closeThread() { document.getElementById('thread-sidebar').style.display = 'none'; }

    function react(id, emoji, path) { if(currentUser) db.ref(`${path}/${id}/reactions/${emoji}/${currentUser.uid}`).set(true); }

    function loadReactions(id, path) {
        db.ref(`${path}/${id}/reactions`).on('value', s => {
            const area = document.getElementById('reac-'+id);
            if(!area) return;
            area.innerHTML = "";
            s.forEach(g => {
                const count = Object.keys(g.val()).length;
                area.innerHTML += `<span class="reac-tag">${g.key} ${count}</span>`;
            });
        });
    }

    function deleteMsg(id, path) { if(confirm("למחוק הודעה?")) db.ref(`${path}/${id}`).remove(); }
    function editMsg(id, old, path) { 
        const n = prompt("ערוך הודעה:", old); 
        if(n) db.ref(`${path}/${id}`).update({text: n}); 
    }
</script>
</body>
</html>
