<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>מערכת נותני שירותים מול היקב</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>

<body class="bg-gradient-to-br from-blue-50 to-indigo-100 min-h-screen p-4">
<div class="max-w-6xl mx-auto">

  <!-- כותרת -->
  <div class="bg-white rounded-lg shadow p-6 mb-6">
    <h1 class="text-3xl font-bold mb-1">מערכת נותני שירותים מול היקב</h1>
    <p class="text-gray-600">מאגר המלצות משותף בזמן אמת</p>
    <p class="text-sm mt-2" id="connectionStatus">🔄 מתחבר…</p>
    <p class="text-sm text-blue-600 mt-1" id="providersCount"></p>
  </div>

  <!-- פאנל -->
  <div class="bg-white rounded-lg shadow p-4 mb-6 flex flex-wrap gap-3 items-center">
    <input id="searchInput" placeholder="חיפוש…" class="border rounded px-3 py-2 flex-1"/>

    <select id="sortSelect" class="border rounded px-3 py-2 text-sm">
      <option value="date_desc">חדש → ישן</option>
      <option value="date_asc">ישן → חדש</option>
      <option value="rating_desc">דירוג גבוה</option>
      <option value="rating_asc">דירוג נמוך</option>
    </select>

    <button onclick="exportCSV()" class="border px-4 py-2 rounded text-sm">
      📥 ייצוא ל-Excel
    </button>

    <button id="addButton" class="bg-blue-600 text-white px-4 py-2 rounded">
      ➕ הוסף
    </button>

    <button id="adminToggle" class="border px-4 py-2 rounded text-sm">
      מצב מנהל
    </button>
  </div>

  <!-- טופס הוספה -->
  <div id="addForm" class="hidden bg-white rounded-lg shadow p-6 mb-6">
    <h2 class="text-xl font-bold mb-4">הוספת נותן שירות</h2>

    <div class="grid md:grid-cols-2 gap-4">
      <input id="name" placeholder="שם מלא *" class="border px-3 py-2 rounded"/>
      <input id="profession" placeholder="מקצוע *" class="border px-3 py-2 rounded"/>
      <input id="phone" placeholder="טלפון" class="border px-3 py-2 rounded"/>
      <input id="recommender" placeholder="ממליץ" class="border px-3 py-2 rounded"/>
      <input id="tags" placeholder="תגיות (חשמלאי, אינסטלציה)" class="border px-3 py-2 rounded md:col-span-2"/>
    </div>

    <textarea id="notes" rows="3" placeholder="הערות" class="border w-full px-3 py-2 rounded mt-3"></textarea>

    <div class="mt-4 flex gap-3">
      <button id="saveButton" class="bg-green-600 text-white px-6 py-2 rounded">שמור</button>
      <button id="cancelButton" class="bg-gray-300 px-6 py-2 rounded">ביטול</button>
    </div>
  </div>

  <!-- רשימה -->
  <div id="providersList" class="grid md:grid-cols-2 gap-6"></div>

</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import { getFirestore, collection, addDoc, deleteDoc, doc, onSnapshot } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyD8foThmQBdKWiW8WsesFpAhaxqBoEO7Y0",
  authDomain: "molhaykev.firebaseapp.com",
  projectId: "molhaykev",
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

let providers = [];
let searchTerm = "";
let sortBy = "date_desc";
let isAdmin = false;

document.getElementById("connectionStatus").textContent = "✅ מחובר";

onSnapshot(collection(db,"providers"), snap => {
  providers = snap.docs.map(d => ({id:d.id, ...d.data()}));
  render();
});

function render(){
  let list = providers.filter(p =>
    p.name?.includes(searchTerm) || p.profession?.includes(searchTerm)
  );

  list.sort((a,b)=>{
    if(sortBy==="rating_desc") return (b.rating||0)-(a.rating||0);
    if(sortBy==="rating_asc") return (a.rating||0)-(b.rating||0);
    if(sortBy==="date_asc") return new Date(a.createdAt)-new Date(b.createdAt);
    return new Date(b.createdAt)-new Date(a.createdAt);
  });

  document.getElementById("providersCount").textContent =
    `סה״כ נותני שירות: ${list.length}`;

  document.getElementById("providersList").innerHTML =
    list.map(p=>`
      <div class="bg-white p-4 rounded shadow">
        <div class="flex justify-between">
          <div>
            <h3 class="font-bold">${p.name}</h3>
            <p class="text-blue-600">${p.profession}</p>
          </div>
          ${isAdmin ? `<button onclick="del('${p.id}')">🗑</button>` : ``}
        </div>

        <p class="text-sm mt-2">${p.notes||""}</p>

        ${p.tags?.length ? `
        <div class="flex gap-2 mt-2 flex-wrap">
          ${p.tags.map(t=>`<span class="bg-blue-100 px-2 py-1 rounded text-xs">${t}</span>`).join("")}
        </div>` : ``}

        <p class="text-xs text-gray-400 mt-2">
          ${new Date(p.createdAt).toLocaleDateString("he-IL")}
        </p>
      </div>
    `).join("");
}

window.del = async id => {
  if(!isAdmin) return alert("מצב מנהל בלבד");
  if(confirm("למחוק?")) await deleteDoc(doc(db,"providers",id));
};

document.getElementById("addButton").onclick = ()=> addForm.classList.toggle("hidden");
document.getElementById("cancelButton").onclick = ()=> addForm.classList.add("hidden");

document.getElementById("saveButton").onclick = async ()=>{
  const tags = tagsInput.value.split(",").map(t=>t.trim()).filter(Boolean);
  await addDoc(collection(db,"providers"),{
    name:name.value, profession:profession.value, phone:phone.value,
    recommender:recommender.value, notes:notes.value, tags,
    createdAt:new Date().toISOString()
  });
  addForm.classList.add("hidden");
};

searchInput.oninput = e => { searchTerm=e.target.value; render(); };
sortSelect.onchange = e => { sortBy=e.target.value; render(); };

adminToggle.onclick = ()=>{
  const p = prompt("סיסמת מנהל");
  if(p==="6969"){ isAdmin=true; adminToggle.textContent="מנהל ✓"; render(); }
};

window.exportCSV = ()=>{
  const rows = providers.map(p=>[p.name,p.profession,p.phone,p.notes,(p.tags||[]).join("|")]);
  const csv = ["שם,מקצוע,טלפון,הערות,תגיות",...rows.map(r=>r.join(","))].join("\n");
  const a=document.createElement("a");
  a.href=URL.createObjectURL(new Blob([csv]));
  a.download="providers.csv";
  a.click();
};

window.addEventListener("offline",()=>connectionStatus.textContent="⚠ אין אינטרנט");
window.addEventListener("online",()=>connectionStatus.textContent="✅ חיבור פעיל");
</script>
</body>
</html>
