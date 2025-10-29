# vocab-test1
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>教師モード - 単語テスト管理</title>
<style>
body { font-family:sans-serif; margin:30px; background:#f7f7f7; }
.container { background:#fff; padding:20px; border-radius:10px; max-width:600px; margin:auto; }
input, textarea, button { width:100%; margin:5px 0; padding:8px; }
button { cursor:pointer; }
.unit { background:#e3f2fd; padding:10px; border-radius:6px; margin-top:10px; }
a { color:#1565c0; text-decoration:none; }
</style>
</head>
<body>
<div class="container">
<h1>📘 教師モード</h1>
<input id="unitName" placeholder="単元名（例：Lesson 1）">
<textarea id="wordInput" placeholder="1行に「apple=りんご」の形式で入力"></textarea>
<button onclick="saveUnit()">💾 単元を保存</button>

<h3>登録済みの単元</h3>
<div id="unitList"></div>

<hr>
<h3>生徒用リンク</h3>
<a href="student.html" target="_blank">▶ 生徒モードを開く</a>
</div>

<script>
let units={};

function loadUnits(){
  const data = localStorage.getItem("vocabUnits");
  if(data) units = JSON.parse(data);
}

function saveUnit(){
  const name = document.getElementById("unitName").value.trim();
  const text = document.getElementById("wordInput").value.trim();
  if(!name || !text) return alert("単元名と単語を入力してください。");
  const lines = text.split("\n").map(line=>{
    const [en,jp] = line.split("=").map(v=>v.trim());
    return {en,jp};
  });
  units[name] = lines;
  localStorage.setItem("vocabUnits",JSON.stringify(units));
  alert(`「${name}」を保存しました！`);
  document.getElementById("unitName").value="";
  document.getElementById("wordInput").value="";
  showUnits();
}

function deleteUnit(name){
  if(confirm(`「${name}」を削除しますか？`)){
    delete units[name];
    localStorage.setItem("vocabUnits",JSON.stringify(units));
    showUnits();
  }
}

function showUnits(){
  const div=document.getElementById("unitList");
  if(Object.keys(units).length===0){
    div.innerHTML="登録された単元はありません。";
    return;
  }
  div.innerHTML=Object.keys(units)
    .map(name=>`
      <div class="unit">
      <b>${name}</b>（${units[name].length}語）
      <button onclick="deleteUnit('${name}')">削除</button><br>
      ${units[name].map(w=>`${w.en} = ${w.jp}`).join("<br>")}
      </div>
    `).join("");
}

loadUnits();
showUnits();
</script>
</body>
</html>


<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>生徒モード - 単語テスト</title>
<style>
body { font-family:sans-serif; margin:30px; background:#eef7ee; }
.container { background:#fff; padding:20px; border-radius:10px; max-width:600px; margin:auto; }
select, input, button { width:100%; margin:5px 0; padding:8px; }
#result { margin-top:10px; font-weight:bold; }
.wrong-list { margin-top:15px; background:#fff3e0; padding:10px; border-radius:6px; }
</style>
</head>
<body>
<div class="container">
<h1>✏️ 生徒モード</h1>

<select id="unitSelect"></select>
<button onclick="startQuiz()">テスト開始</button>

<div id="quizArea" style="display:none;">
  <h2 id="quizWord"></h2>
  <input id="answer" placeholder="英単語を入力してEnterキー" onkeydown="if(event.key==='Enter') checkAnswer()">
  <button onclick="checkAnswer()">答える</button>
  <p id="result"></p>
  <div id="wrongArea" class="wrong-list" style="display:none;"></div>
</div>
</div>

<script>
let units={};
let quizWords=[];
let currentWord;
let incorrect=[];

function loadUnits(){
  const data = localStorage.getItem("vocabUnits");
  if(!data){ alert("単元が見つかりません（教師に確認してください）"); return; }
  units = JSON.parse(data);
  const select = document.getElementById("unitSelect");
  select.innerHTML = Object.keys(units)
    .map(u=>`<option value="${u}">${u}</option>`).join("");
}

function startQuiz(){
  const name = document.getElementById("unitSelect").value;
  quizWords = [...units[name]];
  incorrect = [];
  document.getElementById("quizArea").style.display="block";
  document.getElementById("wrongArea").style.display="none";
  nextQuestion();
}

function nextQuestion(){
  document.getElementById("result").innerText="";
  document.getElementById("answer").value="";
  if(quizWords.length===0) return endQuiz();
  currentWord = quizWords[Math.floor(Math.random()*quizWords.length)];
  document.getElementById("quizWord").innerText=`「${currentWord.jp}」の英語は？`;
}

function checkAnswer(){
  const ans = document.getElementById("answer").value.trim();
  if(ans==="") return;
  const result = document.getElementById("result");
  if(ans.toLowerCase()===currentWord.en.toLowerCase()){
    result.innerHTML="✅ 正解！";
  }else{
    result.innerHTML=`❌ 正解は「${currentWord.en}」`;
    incorrect.push(currentWord);
  }
  quizWords = quizWords.filter(w=>w!==currentWord);
  setTimeout(nextQuestion,800);
}

function endQuiz(){
  const result = document.getElementById("result");
  if(incorrect.length===0){
    result.innerHTML="🎉 全問正解！おめでとうございます！";
  }else{
    result.innerHTML=`終了！間違えた単語は ${incorrect.length} 個`;
    const wrongDiv = document.getElementById("wrongArea");
    wrongDiv.style.display="block";
    wrongDiv.innerHTML="<b>❌ 間違えた単語一覧：</b><br>"+
      incorrect.map(w=>`${w.jp} → ${w.en}`).join("<br>");
  }
}

loadUnits();
</script>
</body>
</html>


