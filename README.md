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
<h1>チケカン</h1>
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

// ランダム色生成
function randomColor() {
  const colors = ["#ffb3ba", "#bae1ff", "#baffc9", "#ffffba", "#ffdfba", "#d5baff"];
  return colors[Math.floor(Math.random() * colors.length)];
}

// カウント調整
function adjustCount(value) {
  count += value;
  if (count < 0) count = 0;
  document.getElementById("ticketCount").textContent = count;

  // 数字色をランダムに変える
  document.getElementById("ticketCount").style.color = randomColor();

  // 音
  const beep = new Audio("https://freesound.org/data/previews/66/66717_931655-lq.mp3");
  beep.play();

  // 履歴に追加
  const tr = document.createElement("tr");
  const now = new Date().toLocaleString();
  tr.innerHTML = `<td>${now}</td><td>${value > 0 ? "+" + value : value}</td><td>${count}</td>`;
  document.getElementById("historyBody").prepend(tr);
}
</script>
</body>
</html>
