<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8" />
<title>チケカク</title>
<style>
body { font-family: sans-serif; padding: 20px; max-width: 700px; margin: auto; background: #e7f7ff; color: #333; }
input, button { width: 100%; padding: 6px; margin: 4px 0; }
h1, h2 { text-align: center; }
#currentDate { text-align: center; font-size: 14px; margin-bottom: 10px; }
#status { margin: 10px 0; color: green; text-align: center; }
#settingsBtn, #closeSettingsBtn { position: fixed; top:5px; right:0; font-size:14px; padding:8px 12px; background:#007acc; color:#fff; border:none; border-radius:6px; z-index:1000; }
#closeSettingsBtn { display:none; }
#settingsPanel { display:none; position:fixed; top:0; left:0; width:100vw; height:100vh; background:#e7f7ff; overflow-y:auto; z-index:999; }
#settingsContent { max-width:700px; margin:auto; padding:20px; }
table { width:100%; border-collapse:collapse; margin-top:20px; }
th,td { border:1px solid #ccc; padding:6px; text-align:center; }
.hidden { display:none; }
</style>
</head>
<body>
<h2>もうチケットをいちいち数えなくていい！</h2>
<h1>チケカク</h1>
<div id="currentDate"></div>
<div id="loginSection">
  <input id="userNumber" placeholder="ユーザー番号">
  <button onclick="signIn()">サインイン</button>
  <button onclick="showSignup()">サインアップ</button>
</div>
<div id="signupSection" class="hidden">
  <input id="signupUserNumber" placeholder="ユーザー番号">
  <input id="signupNickname" placeholder="ニックネーム">
  <button onclick="signUp()">登録する</button>
  <button onclick="showLogin()">戻る</button>
</div>
<div id="mainSection" class="hidden">
  <button id="settingsBtn" onclick="toggleSettings(true)">設定</button>
  <div>
    <strong>ニックネーム:</strong> <span id="nicknameDisplay"></span><br>
    <strong>現在のチケット枚数:</strong> <span id="ticketCount">0</span><br>
    <strong>合計チケット:</strong> <span id="totalTickets">0</span>
  </div>
  <label>使用チケット枚数:</label>
  <input type="number" id="usedTickets" value="0">
  <label>追加チケット枚数:</label>
  <input type="number" id="addedTickets" value="0">
  <button onclick="updateTickets()">記録する</button>
  <table>
    <thead>
      <tr>
        <th>日付</th>
        <th>使用チケット枚数</th>
        <th>追加チケット枚数</th>
        <th>合計チケット</th>
      </tr>
    </thead>
    <tbody id="historyList"></tbody>
  </table>
  <button style="margin-top:20px;" onclick="logout()">ログアウト</button>
</div>
<div id="settingsPanel">
  <div id="settingsContent">
    <button id="closeSettingsBtn" onclick="toggleSettings(false)">戻る</button>
    <h1>設定</h1>
    <label>日付:</label>
    <input type="date" id="manualDate">
    <label>使用チケット枚数:</label>
    <input type="number" id="manualUsedTickets" value="0">
    <label>追加チケット枚数:</label>
    <input type="number" id="manualAddedTickets" value="0">
    <button onclick="addManualEntry()">+ 記録を追加</button>
    <hr>
    <label>削除する日付:</label>
    <input type="date" id="deleteDate">
    <button onclick="deleteEntry()">× 記録を削除</button>
  </div>
</div>
<div id="status"></div>
<script>
const apiUrl = "https://script.google.com/macros/s/AKfycbwFYRJZ8TWtuKBvOG9_JKT2g_9laPW7roVobYDDfAl5sm04AiATbRarlYa_sVfshaBN/exec";
let currentUser = "";
let nickname = "";
let ticketCount = 0;
let history = [];

function getTodayJST() {
  const now = new Date();
  now.setHours(now.getHours() + 9);
  return now.toISOString().split("T")[0];
}
document.getElementById("currentDate").textContent = "今日の日付: " + getTodayJST();
document.getElementById("manualDate").setAttribute("max", getTodayJST());
document.getElementById("deleteDate").setAttribute("max", getTodayJST());

function showSignup() {
  document.getElementById("loginSection").classList.add("hidden");
  document.getElementById("signupSection").classList.remove("hidden");
}
function showLogin() {
  document.getElementById("signupSection").classList.add("hidden");
  document.getElementById("loginSection").classList.remove("hidden");
}
function signIn() {
  const user = document.getElementById("userNumber").value.trim();
  if (!user) {
    alert("ユーザー番号を入力してください");
    return;
  }
  fetch(apiUrl + "?user=" + encodeURIComponent(user))
    .then(res => res.text())
    .then(txt => {
      if (!txt || txt === "{}") {
        alert("ユーザーが存在しません");
        return;
      }
      currentUser = user;
      const data = JSON.parse(txt);
      history = data.history || [];
      nickname = data.nickname || "";
      document.getElementById("nicknameDisplay").textContent = nickname;
      document.getElementById("loginSection").classList.add("hidden");
      document.getElementById("signupSection").classList.add("hidden");
      document.getElementById("mainSection").classList.remove("hidden");
      saveData();
      document.getElementById("status").textContent = "サインインしました";
    });
}
function signUp() {
  const user = document.getElementById("signupUserNumber").value.trim();
  const nick = document.getElementById("signupNickname").value.trim();
  if (!user || !nick) {
    alert("ユーザー番号とニックネームを入力してください");
    return;
  }
  fetch(apiUrl + "?user=" + encodeURIComponent(user) + "&mode=signup&nickname=" + encodeURIComponent(nick), {
    method: "POST",
    body: ""
  })
    .then(res => res.text())
    .then(txt => {
      const data = JSON.parse(txt);
      if (data.error) {
        alert(data.error);
        return;
      }
      alert("サインアップが完了しました。ログインしてください。");
      showLogin();
    });
}
function saveToServer() {
  if (!currentUser) return;
  const content = JSON.stringify(history);
  fetch(apiUrl + "?user=" + encodeURIComponent(currentUser) + "&mode=save", {
    method: "POST",
    body: content
  })
    .then(res => res.text())
    .then(() => {
      document.getElementById("status").textContent = "保存しました";
    });
}
function updateDisplay() {
  document.getElementById("ticketCount").textContent = ticketCount;
  document.getElementById("totalTickets").textContent = ticketCount;
}
function rebuildTicketCount() {
  ticketCount = 0;
  history.sort((a, b) => a.time.localeCompare(b.time));
  history.forEach(e => {
    ticketCount += e.added - e.used;
    e.total = ticketCount;
  });
}
function saveData() {
  rebuildTicketCount();
  updateDisplay();
  refreshHistory();
  saveToServer();
}
function updateTickets() {
  const used = parseInt(document.getElementById("usedTickets").value) || 0;
  const added = parseInt(document.getElementById("addedTickets").value) || 0;
  if (used === 0 && added === 0) {
    alert("使用または追加チケット枚数を入力してください");
    return;
  }
  const now = getTodayJST();
  const idx = history.findIndex(e => e.time === now);
  const entry = { time: now, used, added };
  if (idx !== -1) history[idx] = entry;
  else history.push(entry);
  saveData();
}
function addManualEntry() {
  const time = document.getElementById("manualDate").value;
  if (!time) {
    alert("日付を選んでください");
    return;
  }
  const today = getTodayJST();
  if (time > today) {
    alert("今日より後の日付は選べません");
    return;
  }
  const used = parseInt(document.getElementById("manualUsedTickets").value) || 0;
  const added = parseInt(document.getElementById("manualAddedTickets").value) || 0;
  if (used === 0 && added === 0) {
    alert("使用または追加チケット枚数を入力してください");
    return;
  }
  const idx = history.findIndex(e => e.time === time);
  const entry = { time, used, added };
  if (idx !== -1) history[idx] = entry;
  else history.push(entry);
  saveData();
}
function deleteEntry() {
  const delTime = document.getElementById("deleteDate").
