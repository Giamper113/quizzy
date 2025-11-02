# quizzy
<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Quizzy 🎮 Das interaktive Quiz</title>

<!-- Styling -->
<style>
body {
  font-family: Arial, sans-serif;
  background: #f0f0f0;
  display: flex;
  justify-content: center;
  padding: 50px;
  overflow-x: hidden;
}
#chatbox {
  width: 500px;
  background: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.1);
  position: relative;
}
#messages {
  height: 350px;
  overflow-y: auto;
  border: 1px solid #ddd;
  padding: 10px;
  margin-bottom: 10px;
}
.bot { color: blue; margin: 5px 0; animation: fadeIn 0.3s ease; }
.user { color: green; margin: 5px 0; text-align: right; animation: fadeIn 0.3s ease; }
#choices, #powerups { margin-top: 10px; display: flex; flex-wrap: wrap; gap: 10px; }
.choice-btn, .power-btn { padding: 10px; border: none; border-radius: 5px; cursor: pointer; flex:1; min-width:100px; transition: transform 0.1s; }
.choice-btn { background-color:#2196F3; color:white; }
.choice-btn:hover { background-color:#0b7dda; }
.power-btn { background-color:#FF9800; color:white; }
.power-btn:hover { background-color:#e68a00; }
.choice-btn:active, .power-btn:active { transform: scale(0.95); }
#score, #highscores, #level { margin-top: 10px; font-weight: bold; }
#timerContainer { width:100%; background-color:#ddd; height:10px; border-radius:5px; margin-top:5px; }
#timerBar { height:10px; background-color:#4caf50; width:0%; border-radius:5px; transition: width 0.1s linear, background-color 0.3s; }
.confetti { position:absolute; width:10px; height:10px; top:0; z-index:1000; opacity:0.8; animation:fall 2s linear forwards; }
@keyframes fall { 0% {transform:translateY(0) rotate(0deg);} 100% {transform:translateY(400px) rotate(360deg); opacity:0;} }
@keyframes fadeIn { from {opacity:0;} to {opacity:1;} }
</style>
</head>

<body>
<div id="chatbox">
  <div id="messages">
    <div class="bot">👋 Hallo! Ich bin <b>Quizzy</b> – dein interaktives Quiz!<br>Schreib "Quiz" oder wähle eine Kategorie: Natur 🌿, Geschichte 🏰, Allgemein 🌍.</div>
  </div>
  <div id="timerContainer"><div id="timerBar"></div></div>
  <div id="choices"></div>
  <div id="powerups"></div>
  <div id="score">Punkte: 0</div>
  <div id="level">Level: 1</div>
  <div id="highscores">Highscore: 0</div>
</div>

<!-- Sounds -->
<audio id="correctSound" src="https://www.soundjay.com/buttons/sounds/button-3.mp3"></audio>
<audio id="wrongSound" src="https://www.soundjay.com/buttons/sounds/button-10.mp3"></audio>
<audio id="highscoreSound" src="https://www.soundjay.com/misc/sounds/bell-ringing-05.mp3"></audio>
<audio id="levelUpSound" src="https://www.soundjay.com/misc/sounds/bell-ringing-04.mp3"></audio>

<!-- JavaScript -->
<script>
const messages = document.getElementById("messages");
const choicesDiv = document.getElementById("choices");
const powerDiv = document.getElementById("powerups");
const scoreDiv = document.getElementById("score");
const levelDiv = document.getElementById("level");
const highscoreDiv = document.getElementById("highscores");
const timerBar = document.getElementById("timerBar");
const chatbox = document.getElementById("chatbox");

const correctSound = document.getElementById("correctSound");
const wrongSound = document.getElementById("wrongSound");
const highscoreSound = document.getElementById("highscoreSound");
const levelUpSound = document.getElementById("levelUpSound");

let score=0, level=1, currentAnswer=null, timerInterval=null, timeLeft=20, pointBonus=0;
let highscore = localStorage.getItem("quizzyHighscore") || 0;

highscoreDiv.textContent=`Highscore: ${highscore}`;
levelDiv.textContent=`Level: ${level}`;

const questions = {
  Natur:[{q:"Welches Tier ist das größte auf der Erde?", a:"Blauwal", options:["Elefant","Blauwal","Giraffe","Hai"]},
         {q:"Wie viele Farben hat ein Regenbogen?", a:"7", options:["5","6","7","8"]}],
  Geschichte:[{q:"In welchem Jahr landete der erste Mensch auf dem Mond?", a:"1969", options:["1965","1969","1972","1970"]}],
  Allgemein:[{q:"Was ist die Hauptstadt von Frankreich?", a:"Paris", options:["Paris","Berlin","Rom","Madrid"]}]
};

function addMessage(sender,text,type=""){
  const div=document.createElement("div");
  div.className=sender+(type? " "+type:"");
  div.innerHTML=text;
  messages.appendChild(div);
  messages.scrollTop=messages.scrollHeight;
}

function startQuiz(category){
  stopTimer(); powerDiv.innerHTML="";
  const random=questions[category][Math.floor(Math.random()*questions[category].length)];
  currentAnswer={question:random.q,answer:random.a};
  addMessage("bot",`Kategorie: ${category} 🏆 Frage: ${random.q} (20 Sekunden!)`);
  displayChoices(random.options);
  timeLeft=20; timerBar.style.width="100%"; timerBar.style.backgroundColor="#4caf50";
  timerInterval=setInterval(updateTimer,1000);
}

function displayChoices(options){
  choicesDiv.innerHTML="";
  options.sort(()=>Math.random()-0.5);
  options.forEach(option=>{
    const btn=document.createElement("button");
    btn.className="choice-btn";
    btn.textContent=option;
    btn.onclick=()=>handleAnswer(option);
    choicesDiv.appendChild(btn);
  });
}

function handleAnswer(selected){
  if(!currentAnswer) return; stopTimer(); choicesDiv.innerHTML=""; powerDiv.innerHTML="";
  if(selected===currentAnswer.answer){
    let pointsEarned=1+pointBonus; pointBonus=0;
    addMessage("bot",`🎉 Richtig! +${pointsEarned} Punkt(e)`,"correct"); triggerConfetti(); correctSound.play();
    score+=pointsEarned; scoreDiv.textContent=`Punkte: ${score}`; checkLevelUp();
    if(score>highscore){ highscore=score; highscoreDiv.textContent=`Highscore: ${highscore}`; localStorage.setItem("quizzyHighscore",highscore); addMessage("bot","🏆 Neuer Highscore! 🎊","correct"); triggerConfetti(); highscoreSound.play();}
  } else { addMessage("bot",`❌ Falsch! Die richtige Antwort: ${currentAnswer.answer}`,"wrong"); wrongSound.play();}
  currentAnswer=null;
}

function updateTimer(){timeLeft--; timerBar.style.width=(timeLeft/20*100)+"%"; if(timeLeft<=5) timerBar.style.backgroundColor="red"; if(timeLeft<=0) stopTimer();}
function stopTimer(){clearInterval(timerInterval); timerBar.style.width="0%";}
function checkLevelUp(){let newLevel=Math.floor(score/5)+1; if(newLevel>level){level=newLevel; levelDiv.textContent=`Level: ${level}`; addMessage("bot",`🎉 Level Up! Du bist jetzt Level ${level}! 🎉`,"correct"); levelUpSound.play(); triggerConfetti();}}
function triggerConfetti(){for(let i=0;i<30;i++){const confetti=document.createElement("div"); confetti.className="confetti"; confetti.style.backgroundColor=`hsl(${Math.random()*360},100%,50%)`; confetti.style.left=Math.random()*480+"px"; confetti.style.animationDuration=(2+Math.random()*2)+"s"; chatbox.appendChild(confetti); setTimeout(()=>chatbox.removeChild(confetti),4000);}}
</script>
</body>
</html>
