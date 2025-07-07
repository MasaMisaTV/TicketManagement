<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8" />
<title>チケカン - チケットカウンター</title>
<style>
body {
  font-family: sans-serif;
  margin: 0;
  padding: 0;
  background: #f0f0f0;
  display: flex;
  flex-direction: column;
  align-items: center;
}
h1 {
  margin: 20px;
}
#ticketCount {
  font-size: 80px;
  margin: 20px;
}
.buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
  margin-bottom: 20px;
}
.buttons button {
  font-size: 24px;
  padding: 20px 30px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}
#history {
  width: 90%;
  max-width: 500px;
  background: #fff;
  border-collapse: collapse;
}
#history th, #history td {
  border: 1px solid #ccc;
  padding: 6px;
  text-align: center;
}
</style>
</head>
<body>
<h1>🎫 チケカン</h1>
<div id="ticketCount">0</div>
<div class="buttons">
  <button onclick="adjustCount(2)">+2</button>
  <button onclick="adjustCount(3)">+3</button>
  <button onclick="adjustCount(4)">+4</button>
  <button onclick="adjustCount(-2)">-2</button>
  <button onclick="adjustCount(-3)">-3</button>
  <button onclick="adjustCount(-4)">-4</button>
</div>
<table id="history">
  <thead>
    <tr><th>日時</th><th>増減</th><th>残数</th></tr>
  </thead>
  <tbody id="historyBody"></tbody>
</table>

<script>
// チケット枚数
let count = 0;

// ページ読み込み時にローカルから復元
window.onload = function() {
  const savedCount = localStorage.getItem("ticketCount");
  const savedHistory = localStorage.getItem("ticketHistory");
  if (savedCount !== null) {
    count = parseInt(savedCount);
    document.getElementById("ticketCount").textContent = count;
  }
  if (savedHistory) {
    document.getElementById("historyBody").innerHTML = savedHistory;
  }
};

// カウント調整
function adjustCount(value) {
  count += value;
  if (count < 0) count = 0;
  document.getElementById("ticketCount").textContent = count;

  // 履歴を表示
  const tr = document.createElement("tr");
  const now = new Date().toLocaleString();
  tr.innerHTML = `<td>${now}</td><td>${value > 0 ? "+" + value : value}</td><td>${count}</td>`;
  document.getElementById("historyBody").prepend(tr);

  // ローカルに保存
  localStorage.setItem("ticketCount", count);
  localStorage.setItem("ticketHistory", document.getElementById("historyBody").innerHTML);

  // スプレッドシートに送信
  fetch("https://script.google.com/macros/s/AKfycbwhibx2kVVflaGjF6WyqXyu3GVL0Xcs3qmUAkBXDqNrjmfiwhBhk1a4DOvvH5tWKoPrRA/exec", {
    method: "POST",
    body: JSON.stringify({
      change: value,
      total: count
    }),
    headers: { "Content-Type": "application/json" }
  }).catch(error => {
    console.error("送信エラー:", error);
  });
}
</script>
</body>
</html>
