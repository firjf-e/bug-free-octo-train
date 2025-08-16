# bug-free-octo-train
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>보상형 할일 게임 – 신지성 시스템</title>
<style>
  body{font-family:system-ui,Segoe UI,Apple SD Gothic Neo,Roboto,Arial; margin:0; background:#0b1015; color:#e6edf3}
  header{display:flex; gap:12px; align-items:center; justify-content:space-between; padding:14px 16px; border-bottom:1px solid #1f2937; background:#0f1620; position:sticky; top:0; z-index:5}
  .brand{font-weight:700}
  .row{display:flex; gap:16px; padding:16px; flex-wrap:wrap}
  .col{flex:1 1 360px}
  .card{background:#111827; border:1px solid #1f2937; border-radius:16px; padding:16px; box-shadow:0 10px 30px rgba(0,0,0,.25)}
  .title{font-weight:700; margin-bottom:10px}
  .muted{color:#94a3b8}
  button{background:#2563eb; color:white; border:0; border-radius:12px; padding:10px 14px; cursor:pointer}
  button.ghost{background:transparent; color:#93c5fd; border:1px solid #1f2937}
  button.warn{background:#ef4444}
  button:disabled{opacity:.6; cursor:not-allowed}
  input,select{background:#0b1220; color:#e6edf3; border:1px solid #1f2937; border-radius:10px; padding:10px}
  .grid{display:grid; gap:10px}
  .grid2{display:grid; grid-template-columns:1fr 1fr; gap:10px}
  .chip{display:inline-flex; align-items:center; gap:6px; padding:6px 10px; border-radius:999px; background:#0b1220; border:1px solid #1f2937; font-size:14px}
  .list{display:flex; flex-direction:column; gap:8px}
  .task{display:flex; justify-content:space-between; align-items:center; gap:10px; padding:10px; border-radius:12px; background:#0c1423; border:1px solid #1f2937}
  .tag{font-size:12px; color:#93c5fd}
  .kicker{font-size:12px; color:#9ca3af}
  .rowwrap{display:flex; gap:8px; flex-wrap:wrap}
  .foods{display:grid; grid-template-columns:repeat(3,1fr); gap:8px}
  .food{padding:10px; border-radius:12px; background:#0c1423; border:1px solid #1f2937; display:flex; justify-content:space-between; align-items:center}
  .pill{font-size:12px; padding:4px 8px; border-radius:999px; border:1px solid #1f2937; background:#0b1220}
  .danger{color:#fca5a5}
  .good{color:#86efac}
  .mono{font-family:ui-monospace, SFMono-Regular, Menlo, monospace; font-size:13px}
  .right{float:right}
  .hidden{display:none}
  footer{padding:20px; text-align:center; color:#94a3b8}
</style>
</head>
<body>
<header>
  <div class="brand">✅ 할일-보상 게임</div>
  <div class="rowwrap">
    <button id="exportBtn" class="ghost">데이터 내보내기</button>
    <label class="chip">
      JSON 가져오기
      <input type="file" id="importFile" accept="application/json" />
    </label>
  </div>
</header>

<div class="row">
  <div class="col">
    <div class="card">
      <div class="title">프로필</div>
      <div class="grid">
        <div class="rowwrap">
          <span class="chip">아바타: <span id="avatar">😐</span></span>
          <span class="chip">이름: <span id="name">신지성</span></span>
          <span class="chip">레벨: <span id="level">Vb.1</span></span>
        </div>
        <div class="rowwrap">
          <span class="chip">코인: <b id="coins">0</b></span>
          <span class="chip">에메랄드: <b id="emeralds">0</b></span>
          <span class="chip">Aura: <b id="aura">0</b></span>
        </div>
        <div class="grid2">
          <input id="nameInput" placeholder="이름 변경(엔터)" />
          <button id="resetBtn" class="warn">전체 초기화</button>
        </div>
        <div class="kicker">보상차단: <span id="blockDays">0</span>일 / 정지: <span id="banDays">0</span>일</div>
      </div>
    </div>

    <div class="card">
      <div class="title">할일 추가</div>
      <div class="grid2">
        <input id="taskInput" placeholder='예) "약,세수,로션" 또는 "11시에 잠들기"' />
        <button id="addTaskBtn">할일저장</button>
      </div>
      <div class="kicker">특수패턴 자동 적용: “11시에 잠들기”, “라면 2번 절대 먹지 말기”</div>
    </div>

    <div class="card">
      <div class="title">라면 위반 카운터</div>
      <div class="grid2">
        <div class="rowwrap">
          <button id="ramenPlus">라면 +1</button>
          <button id="ramenZero" class="ghost">카운터 초기화</button>
        </div>
        <div class="kicker">오늘 라면 횟수: <b id="ramenCount">0</b> (2회 달성 시 자동 패널티: Aura -3)</div>
      </div>
    </div>

    <div class="card">
      <div class="title">규칙 위반 테스트(철퇴 버튼)</div>
      <div class="grid2">
        <button id="cheatAura" class="warn">1) 오라 무단 사용</button>
        <button id="cheatCoin" class="warn">2) 코인 무단 조작</button>
      </div>
      <div class="grid" style="margin-top:8px">
        <button id="cheatTask" class="warn">3) 할일 결과 조작</button>
      </div>
      <div class="kicker">실운영에선 숨겨도 되고, 여기서는 규칙 작동 확인용.</div>
    </div>
  </div>

  <div class="col">
    <div class="card">
      <div class="title">할일 리스트</div>
      <div id="taskList" class="list"></div>
    </div>

    <div class="card">
      <div class="title">보상 – 랜덤 음식 6종 세트</div>
      <div id="rewardBox" class="foods"></div>
      <div class="kicker">할일 완료 시 자동 생성됨.</div>
    </div>

    <div class="card">
      <div class="title">🛒 쇼핑창</div>
      <div id="shop" class="foods"></div>
      <div class="kicker">선택 시 “~코인이 필요합니다. 지불하시겠습니까?” 확인 → 레시피 공개</div>
      <div id="recipeBox" class="mono"></div>
    </div>
  </div>
</div>

<footer>© 보상형 할일 게임 • 깃허브 Pages 배포용</footer>

<script>
const AVATARS = ["😐","🤨","🤔","😙","😏","🤑"];
const LEVELS  = ["Vb.1","Vb.2","Vb.3","Vb.4","Vb.5","Vb.6"];
const AURA_THRESH = [0,50,120,210,320,450]; // 각 단계 최소 오라

// 음식 카테고리 & 레시피(일부 고급요리)
const FOODS = {
  "국류":[
    {name:"된장국", premium:false, recipe:["물+멸치육수","된장 풀기","두부/호박 넣고 끓이기"]},
    {name:"미역국", premium:false, recipe:["미역 불리기","참기름에 볶기","물+간장 살짝 끓이기"]},
    {name:"육개장", premium:true, recipe:["소고기/파 볶기","고춧가루/고사리","오래 끓여 깊은맛"]}
  ],
  "고기류":[
    {name:"불고기", premium:false, recipe:["간장+설탕+마늘","소고기 재우기","센불 볶기"]},
    {name:"삼겹살", premium:false, recipe:["소금간","노릇하게 굽기","쌈장이랑"]},
    {name:"닭갈비", premium:true, recipe:["고추장 양념","닭/양배추 볶기","떡사리 선택"]}
  ],
  "밥류":[
    {name:"김치볶음밥", premium:false, recipe:["김치 먼저 볶기","밥 넣고 섞기","참기름 마무리"]},
    {name:"참치마요덮밥", premium:false, recipe:["참치+마요","간장 약간","밥 위에 올리기"]},
    {name:"비빔밥", premium:true, recipe:["나물/고기 토핑","고추장 소스","비벼먹기"]}
  ],
  "면류":[
    {name:"우동", premium:false, recipe:["가쓰오육수","우동면","쪽파"]},
    {name:"라멘", premium:true, recipe:["돈코츠/쇼유 베이스","면 삶기","차슈/계란"]},
    {name:"스파게티", premium:true, recipe:["면 삶기","올리브유/마늘","소스 합체"]}
  ],
  "빵·간식류":[
    {name:"크루아상", premium:true, recipe:["버터 레이어링","숙성","굽기"]},
    {name:"호떡", premium:false, recipe:["반죽","설탕/씨앗","노릇하게"]},
    {name:"샌드위치", premium:false, recipe:["빵","햄치즈/야채","소스"]}
  ],
  "해산물류":[
    {name:"연어초밥", premium:true, recipe:["밥 식초간","연어 올리기","와사비 선택"]},
    {name:"새우튀김", premium:false, recipe:["반죽","튀기기","소금/소스"]},
    {name:"조개탕", premium:true, recipe:["해감","맑게 끓이기","파/마늘"]}
  ]
};

const BASE_PRICE = 10; // 기본 10코인
function priced(item){
  if(item.premium){
    // 20~30% 인상: 12 또는 13
    return Math.random()<0.5 ? 12 : 13;
  }
  return BASE_PRICE;
}

const state = load() || {
  name:"신지성",
  levelIdx:0,
  coins:40,
  emeralds:0,
  aura:30,
  banDays:0,
  blockDays:0,
  ramenCount:0,
  items:[], // 해금 레시피 목록
  tasks:[
    {id:id(), title:"걷기 5,000~6,000걸음 (8/18)", done:false},
    {id:id(), title:"매일 적당히 먹기", done:false},
    {id:id(), title:"아무 음악이나 30분 틀기", done:false},
    {id:id(), title:"11시에 잠들기", done:false, special:"sleep11"},
    {id:id(), title:"윗몸일으키기 아침·저녁 20회", done:false},
    {id:id(), title:"라면 2번 절대 먹지 말기", done:false, special:"ramen2"},
  ],
  lastReward:[] // 최근 생성된 6개 보상
};

function id(){ return Math.random().toString(36).slice(2,9); }
function save(){ localStorage.setItem("questGame", JSON.stringify(state)); }
function load(){ try{ return JSON.parse(localStorage.getItem("questGame")); }catch(e){ return null; } }

function syncLevel(){
  // 오라 기준 자동 레벨
  let idx = 0;
  for(let i=0;i<AURA_THRESH.length;i++){
    if(state.aura>=AURA_THRESH[i]) idx = i;
  }
  state.levelIdx = idx;
  if(state.levelIdx<0) state.levelIdx=0;
}

function renderProfile(){
  syncLevel();
  document.getElementById("avatar").textContent = AVATARS[state.levelIdx];
  document.getElementById("name").textContent   = state.name;
  document.getElementById("level").textContent  = LEVELS[state.levelIdx];
  document.getElementById("coins").textContent  = state.coins;
  document.getElementById("emeralds").textContent = state.emeralds;
  document.getElementById("aura").textContent   = state.aura;
  document.getElementById("blockDays").textContent = state.blockDays;
  document.getElementById("banDays").textContent = state.banDays;
  document.getElementById("ramenCount").textContent = state.ramenCount;
}

function renderTasks(){
  const box = document.getElementById("taskList");
  box.innerHTML = "";
  state.tasks.forEach(t=>{
    const el = document.createElement("div");
    el.className = "task";
    const left = document.createElement("div");
    left.innerHTML = `<div>${t.title} ${t.special ? `<span class="tag">특수</span>`:""}</div>
      <div class="kicker">${t.done?'<span class="good">완료</span>':'진행중'}</div>`;
    const right = document.createElement("div");
    const doneBtn = document.createElement("button");
    doneBtn.textContent = "완료";
    doneBtn.onclick = ()=> completeTask(t.id);
    const failBtn = document.createElement("button");
    failBtn.textContent = "실패";
    failBtn.className = "ghost";
    failBtn.onclick = ()=> failTask(t.id);
    right.append(doneBtn, failBtn);
    el.append(left, right);
    box.appendChild(el);
  });
}

function randOne(arr){ return arr[Math.floor(Math.random()*arr.length)]; }
function generateReward6(){
  const cats = Object.keys(FOODS);
  state.lastReward = cats.map(cat=>{
    const pick = {...randOne(FOODS[cat])};
    pick.category = cat;
    pick.price = priced(pick);
    return pick;
  });
}

function renderReward(){
  const box = document.getElementById("rewardBox");
  box.innerHTML="";
  state.lastReward.forEach(f=>{
    const el = document.createElement("div");
    el.className="food";
    el.innerHTML = `<div>
      <div><b>${f.name}</b> <span class="pill">${f.category}${f.premium?' · 고급':''}</span></div>
      <div class="kicker">보상 생성됨</div>
    </div>
    <div class="pill">${f.price}코인</div>`;
    box.appendChild(el);
  });
}

function renderShop(){
  const box = document.getElementById("shop");
  box.innerHTML="";
  // 상점은 최근 보상 6개를 그대로 노출
  state.lastReward.forEach((f, idx)=>{
    const el = document.createElement("div");
    el.className = "food";
    const btn = document.createElement("button");
    btn.textContent = "선택";
    btn.onclick = ()=> attemptBuy(idx);
    el.innerHTML = `<div>
      <div><b>${f.name}</b> <span class="pill">${f.category}${f.premium?' · 고급':''}</span></div>
      <div class="kicker">${f.price}코인 필요</div>
    </div>`;
    el.appendChild(btn);
    box.appendChild(el);
  });
}

function attemptBuy(i){
  const f = state.lastReward[i];
  const msg = `${f.name}은(는) ${f.price}코인이 필요합니다. ${f.price}코인을 지불하시겠습니까?`;
  if(!confirm(msg)) return;
  if(state.coins < f.price){ alert("코인이 부족합니다."); return; }
  state.coins -= f.price;
  // 레시피 공개 + 아이템 해금
  if(!state.items.find(x=>x.name===f.name)) state.items.push({name:f.name, category:f.category});
  showRecipe(f);
  save(); renderProfile();
}

function showRecipe(f){
  const box = document.getElementById("recipeBox");
  const steps = (f.recipe||["재료 준비","기본 손질","맛있게 조리"]).map((s,i)=>`${i+1}. ${s}`).join("\n");
  box.textContent = `[${f.category}] ${f.name} 레시피\n${steps}`;
}

function rewardOnSuccess(){
  if(state.blockDays>0 || state.banDays>0){
    // 차단/정지 중이면 코인·오라 보상 없음
    alert("보상 차단/정지 중이라 코인·오라 지급이 중단됨. 음식 리스트만 볼 수 있음.");
  }else{
    state.coins += 10;
    state.aura  += 10;
  }
  generateReward6();
  save();
  renderProfile(); renderReward(); renderShop();
}

function completeTask(taskId){
  const t = state.tasks.find(x=>x.id===taskId);
  if(!t) return;
  t.done = true;
  alert(`잘하네, 너가 가진 코인으로 추천음식 골라봐 ⬇️`);
  rewardOnSuccess();
  renderTasks();
}

function failTask(taskId){
  const t = state.tasks.find(x=>x.id===taskId);
  if(!t) return;
  t.done = false;
  let delta = -1;
  if(t.special==="sleep11") delta = -3;
  state.aura += delta;
  alert("아이고...실패했네..다음에 다시 도전해봐!");
  save(); renderProfile(); renderTasks();
}

function addTask(raw){
  const title = raw.trim();
  if(!title) return;
  const task = {id:id(), title, done:false};
  if(title.includes("11시에 잠들기")) task.special = "sleep11";
  if(title.includes("라면 2번 절대 먹지 말기")) task.special = "ramen2";
  state.tasks.unshift(task);
  save(); renderTasks();
}

function ramenInc(){
  state.ramenCount++;
  if(state.ramenCount>=2){
    // 특수 패널티 발동
    state.aura -= 3;
    alert("라면 2회 달성 → 특수 패널티: Aura -3");
  }
  save(); renderProfile();
}
function ramenReset(){ state.ramenCount=0; save(); renderProfile(); }

function punish(type){
  // 공통 도우미
  function block(days){ state.blockDays = Math.max(state.blockDays, days); }
  function ban(days){ state.banDays   = Math.max(state.banDays, days); }
  function levelDown(n){
    state.levelIdx = Math.max(0, state.levelIdx - n);
    // 레벨 수동 내리면서 오라도 낮은 경계로 보정
    const target = Math.max(0, state.levelIdx-0);
    state.aura = Math.min(state.aura, AURA_THRESH[target]);
  }
  function vbReset(){ state.levelIdx=0; state.aura = Math.min(state.aura, AURA_THRESH[0]); }
  function removeItems(k=2){
    while(k-- >0 && state.items.length>0){
      const idx = Math.floor(Math.random()*state.items.length);
      state.items.splice(idx,1);
    }
  }
  function addPenaltyTasks(){
    addTask("패널티: 푸쉬업 100회");
    addTask("패널티: 걷기 10,000걸음");
  }

  if(type==="aura"){
    ban(10);
    state.aura -= 5;
    state.coins = Math.max(0, state.coins-20);
    vbReset();
    block(5);
    removeItems(2);
    alert("오라 무단 사용 철퇴 발동");
  }else if(type==="coin"){
    ban(7);
    state.coins = Math.max(0, state.coins-30);
    state.aura -= 5;
    levelDown(2);
    block(3);
    alert("코인 무단 조작 철퇴 발동");
  }else if(type==="task"){
    state.aura -= 10;
    state.coins = Math.max(0, state.coins-40);
    levelDown(1);
    addPenaltyTasks();
    alert("할일 결과 조작 철퇴 발동");
  }
  save(); renderProfile(); renderTasks();
}

// 차단/정지 일수 감소(하루 지나면 수동으로 눌러도 됨)
function tickDay(){
  if(state.blockDays>0) state.blockDays--;
  if(state.banDays>0) state.banDays--;
  save(); renderProfile();
}

// 초기 렌더
if(state.lastReward.length===0) generateReward6();
renderProfile(); renderTasks(); renderReward(); renderShop();

// 이벤트
document.getElementById("addTaskBtn").onclick = ()=>{
  const v = document.getElementById("taskInput").value;
  document.getElementById("taskInput").value="";
  addTask(v);
};

document.getElementById("nameInput").addEventListener("keydown", e=>{
  if(e.key==="Enter"){
    const v = e.target.value.trim();
    if(v){ state.name=v; save(); renderProfile(); e.target.value=""; }
  }
});

document.getElementById("resetBtn").onclick = ()=>{
  if(!confirm("정말 전체 초기화할래?")) return;
  localStorage.removeItem("questGame");
  location.reload();
};

document.getElementById("ramenPlus").onclick = ramenInc;
document.getElementById("ramenZero").onclick = ramenReset;

document.getElementById("cheatAura").onclick = ()=>punish("aura");
document.getElementById("cheatCoin").onclick = ()=>punish("coin");
document.getElementById("cheatTask").onclick = ()=>punish("task");

// 내보내기/가져오기
document.getElementById("exportBtn").onclick = ()=>{
  const blob = new Blob([JSON.stringify(state,null,2)], {type:"application/json"});
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "questGame-save.json";
  a.click();
  URL.revokeObjectURL(a.href);
};
document.getElementById("importFile").onchange = (e)=>{
  const f = e.target.files[0];
  if(!f) return;
  const reader = new FileReader();
  reader.onload = ()=>{
    try{
      const data = JSON.parse(reader.result);
      localStorage.setItem("questGame", JSON.stringify(data));
      location.reload();
    }catch(err){ alert("JSON 형식 오류"); }
  };
  reader.readAsText(f);
};

// 하루 경과 수동 버튼(원하면 상단에 배치해도 됨)
// 사용 예: 개발 편의
window.tickDay = tickDay;
</script>
</body>
</html>

