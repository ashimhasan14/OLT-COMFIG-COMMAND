
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>OLT CONFIG</title>
<style>
body{margin:0;background:#0b1220;color:white;font-family:Arial}
header{padding:20px;background:#14243b}
main{max-width:950px;margin:30px auto;padding:15px}
.card{background:#17263d;padding:20px;border-radius:12px;margin-bottom:18px}
input,select,textarea,button{padding:12px;margin:7px 0;width:100%;box-sizing:border-box;border-radius:7px;border:1px solid #344761;background:#0e192b;color:white}
button{background:#24bd9c;color:#071b17;font-weight:bold;cursor:pointer}
pre{white-space:pre-wrap;background:#08101d;padding:12px;border-radius:8px}
.hidden{display:none}
</style>
</head>
<body>
<header>
<h2>OLT CONFIG</h2>
NETWORK COMMAND CENTER
</header>
<main>
<section id="login" class="card">
<h2>Login</h2>
<input id="username" placeholder="Username">
<input id="password" type="password" placeholder="Password">
<button id="loginBtn">Login</button>
<p id="msg"></p>
</section>

<section id="dashboard" class="hidden">
<div class="card">
<h2>Command Center</h2>
<button id="logout">Logout</button>
<select id="brand">
<option value="">All Brands</option>
<option>BDCOM</option><option>VSOL</option>
<option>HUAWEI</option><option>C-DATA</option>
<option>DN OPTIC</option><option>ECOM</option>
<option>OPTILINK</option><option>SWITCH</option>
</select>
<input id="search" placeholder="Search command">
<div id="commands">Loading...</div>
</div>

<div id="admin" class="card hidden">
<h3>Admin - Add Command</h3>
<input id="newBrand" placeholder="Brand">
<input id="newTitle" placeholder="Command Title">
<select id="newType">
<option>Short Command</option>
<option>Full Configuration</option>
<option>Switch Configuration</option>
</select>
<textarea id="newBody" placeholder="Enter command"></textarea>
<button id="save">Save Command</button>
<p id="saveMsg"></p>
</div>
</section>
</main>

<script type="module">
import {initializeApp} from
"https://www.gstatic.com/firebasejs/10.12.5/firebase-app.js";
import {getAuth,signInWithEmailAndPassword,onAuthStateChanged,signOut}
from "https://www.gstatic.com/firebasejs/10.12.5/firebase-auth.js";
import {getFirestore,collection,addDoc,getDocs,doc,getDoc,serverTimestamp}
from "https://www.gstatic.com/firebasejs/10.12.5/firebase-firestore.js";

const firebaseConfig={
apiKey:"AIzaSyBY0CN1z7_1s-2QvN642Yw59RW6AVRm3qQ",
authDomain:"olt-config-51859.firebaseapp.com",
projectId:"olt-config-51859",
storageBucket:"olt-config-51859.firebasestorage.app",
messagingSenderId:"760732296839",
appId:"1:760732296839:web:2c9f52f3d8895d3499fa22",
measurementId:"G-VRQ1SM5V7M"
};

const app=initializeApp(firebaseConfig);
const auth=getAuth(app);
const db=getFirestore(app);
const $=id=>document.getElementById(id);
let commands=[];

$("loginBtn").onclick=async()=>{
try{
const u=$("username").value.trim();
const p=$("password").value;
const email=u==="ashimhasan"?"ashimhasan@olt-config.local":u;
await signInWithEmailAndPassword(auth,email,p);
}catch(e){$("msg").textContent="Login failed. Check username and password."}
};

$("logout").onclick=()=>signOut(auth);

onAuthStateChanged(auth,async user=>{
$("login").classList.toggle("hidden",!!user);
$("dashboard").classList.toggle("hidden",!user);
if(!user)return;
try{
const userDoc=await getDoc(doc(db,"users",user.uid));
const isAdmin=userDoc.exists()&&userDoc.data().role==="admin";
$("admin").classList.toggle("hidden",!isAdmin);
await loadCommands();
}catch(e){$("commands").textContent="Database access error."}
});

async function loadCommands(){
try{
const snap=await getDocs(collection(db,"commands"));
commands=snap.docs.map(d=>({id:d.id,...d.data()}));
render();
}catch(e){$("commands").textContent="Could not load commands."}
}

function render(){
const b=$("brand").value;
const s=$("search").value.toLowerCase();
const rows=commands.filter(x=>
(!b||x.brand===b)&&
(`${x.title} ${x.brand} ${x.body}`.toLowerCase().includes(s))
);
$("commands").innerHTML="";
rows.forEach(x=>{
const box=document.createElement("div");
box.className="card";
const title=document.createElement("b");
title.textContent=x.title+" - "+x.brand;
const pre=document.createElement("pre");
pre.textContent=x.body;
const btn=document.createElement("button");
btn.textContent="Copy Command";
btn.onclick=()=>navigator.clipboard.writeText(x.body);
box.append(title,pre,btn);
$("commands").append(box);
});
if(!rows.length)$("commands").textContent="No commands found.";
}

$("brand").onchange=render;
$("search").oninput=render;

$("save").onclick=async()=>{
try{
await addDoc(collection(db,"commands"),{
brand:$("newBrand").value.trim().toUpperCase(),
title:$("newTitle").value.trim(),
kind:$("newType").value,
body:$("newBody").value.trim(),
createdAt:serverTimestamp()
});
$("saveMsg").textContent="Command saved!";
$("newTitle").value="";
$("newBody").value="";
await loadCommands();
}catch(e){$("saveMsg").textContent="Save failed. Check Admin permission."}
};
</script>
</body>
</html>
