<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>מערכת נותני שירותים מול היקב</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gradient-to-br from-blue-50 to-indigo-100 min-h-screen p-4">
  <div class="max-w-6xl mx-auto">
    <!-- כותרת -->
    <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
      <h1 class="text-3xl font-bold text-gray-800 mb-2">מערכת נותני שירותים מול היקב</h1>
      <p class="text-gray-600">שתף את החוויה שלך ועזור לאחרים למצוא את נותני השירות הטובים ביותר</p>
      <p class="text-sm text-green-600 mt-2" id="connectionStatus">🔄 מתחבר ל-Firebase...</p>
    </div>

    <!-- פאנל ניהול -->
    <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
      <div class="flex flex-wrap gap-4 items-center justify-between">
        <div class="flex-1 min-w-[250px]">
          <div class="relative">
            <span class="absolute right-3 top-3 text-gray-400">🔍</span>
            <input
              type="text"
              id="searchInput"
              placeholder="חפש לפי שם או מקצוע..."
              class="w-full pr-10 pl-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            />
          </div>
        </div>
        <div class="flex gap-3">
          <button id="adminToggle" class="text-sm font-medium border border-gray-300 px-4 py-2 rounded-lg hover:bg-gray-50">
            מצב מנהל
          </button>
          <button id="addButton" class="flex items-center gap-2 bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors">
            ➕ הוסף נותן שירות
          </button>
        </div>
      </div>
    </div>

    <!-- חלון סיסמא -->
    <div id="passwordModal" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg shadow-xl p-6 max-w-md w-full mx-4">
        <h3 class="text-xl font-bold mb-4">הזן סיסמת מנהל</h3>
        <input
          type="password"
          id="adminPassword"
          placeholder="הזן סיסמה"
          class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 mb-4"
        />
        <div class="flex gap-3">
          <button id="passwordSubmit" class="flex-1 bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700">
            אישור
          </button>
          <button id="passwordCancel" class="flex-1 bg-gray-300 text-gray-700 px-4 py-2 rounded-lg hover:bg-gray-400">
            ביטול
          </button>
        </div>
      </div>
    </div>

    <!-- טופס הוספה -->
    <div id="addForm" class="hidden bg-white rounded-lg shadow-lg p-6 mb-6">
      <h2 class="text-xl font-bold mb-4">הוסף נותן שירות חדש</h2>
      <div class="space-y-4">
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div>
            <label class="block text-sm font-medium mb-1">שם מלא *</label>
            <input type="text" id="name" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
          </div>
          <div>
            <label class="block text-sm font-medium mb-1">מקצוע *</label>
            <input type="text" id="profession" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
          </div>
          <div>
            <label class="block text-sm font-medium mb-1">טלפון</label>
            <input type="tel" id="phone" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
          </div>
          <div>
            <label class="block text-sm font-medium mb-1">שם הממליץ</label>
            <input type="text" id="recommender" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
          </div>
        </div>
        <div>
          <label class="block text-sm font-medium mb-1">דירוג</label>
          <input type="hidden" id="rating" value="0">
          <div id="starsDisplay" class="flex gap-1"></div>
        </div>
        <div>
          <label class="block text-sm font-medium mb-1">הערות והמלצות</label>
          <textarea id="notes" rows="4" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500" placeholder="שתף את החוויה שלך..."></textarea>
        </div>
        <div class="flex gap-3">
          <button id="saveButton" class="bg-green-600 text-white px-6 py-2 rounded-lg hover:bg-green-700">
            שמור
          </button>
          <button id="cancelButton" class="bg-gray-300 text-gray-700 px-6 py-2 rounded-lg hover:bg-gray-400">
            ביטול
          </button>
        </div>
      </div>
    </div>

    <!-- רשימת נותני שירות -->
    <div id="providersList" class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <div class="col-span-full text-center py-12 bg-white rounded-lg shadow-lg">
        <p class="text-gray-500 text-lg">טוען נתונים...</p>
      </div>
    </div>

    <!-- הערה -->
    <div class="mt-6 bg-yellow-50 border border-yellow-200 rounded-lg p-4 text-sm text-yellow-800">
      <p class="font-medium">💡 שים לב:</p>
      <p>המידע במערכת זו נשמר בענן של Google (Firebase) ומשותף עם כל המשתמשים בזמן אמת!</p>
    </div>
  </div>

  <script type="module">
    import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js';
    import { getFirestore, collection, addDoc, getDocs, deleteDoc, doc, onSnapshot, query, orderBy } from 'https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js';

    const firebaseConfig = {
      apiKey: "AIzaSyD8foThmQBdKWiW8WsesFpAhaxqBoEO7Y0",
      authDomain: "molhaykev.firebaseapp.com",
      projectId: "molhaykev",
      storageBucket: "molhaykev.firebasestorage.app",
      messagingSenderId: "207007704124",
      appId: "1:207007704124:web:9feb50ac5f897aa9588f39",
      measurementId: "G-RYBM9T3F47"
    };

    let app, db;
    let providers = [];
    let isAdmin = false;
    let searchTerm = '';

    // אתחול Firebase
    try {
      app = initializeApp(firebaseConfig);
      db = getFirestore(app);
      document.getElementById('connectionStatus').textContent = '✅ מחובר ל-Firebase - הנתונים משותפים לכולם!';
      console.log('Firebase initialized successfully');
      
      // האזנה לשינויים בזמן אמת
      const q = query(collection(db, "providers"));
      onSnapshot(q, (snapshot) => {
        providers = [];
        snapshot.forEach((doc) => {
          providers.push({ id: doc.id, ...doc.data() });
        });
        console.log('Loaded providers:', providers.length);
        renderProviders();
      }, (error) => {
        console.error('Error in snapshot listener:', error);
        document.getElementById('connectionStatus').textContent = '❌ שגיאה בחיבור ל-Firebase';
      });
      
    } catch (error) {
      console.error('Firebase initialization error:', error);
      document.getElementById('connectionStatus').textContent = '❌ שגיאה באתחול Firebase';
    }

    // כוכבי דירוג
    function renderStars(rating, editable = false) {
      const container = document.getElementById('starsDisplay');
      container.innerHTML = '';
      for (let i = 1; i <= 5; i++) {
        const star = document.createElement('span');
        star.innerHTML = '★';
        star.className = `text-2xl ${i <= rating ? 'text-yellow-400' : 'text-gray-300'} ${editable ? 'cursor-pointer hover:scale-110 transition-transform' : ''}`;
        if (editable) {
          star.onclick = () => {
            document.getElementById('rating').value = i;
            renderStars(i, true);
          };
        }
        container.appendChild(star);
      }
    }

    // רינדור רשימה
    function renderProviders() {
      const filtered = providers.filter(p =>
        p.name?.toLowerCase().includes(searchTerm.toLowerCase()) ||
        p.profession?.toLowerCase().includes(searchTerm.toLowerCase())
      );

      const container = document.getElementById('providersList');
      
      if (filtered.length === 0) {
        container.innerHTML = `
          <div class="col-span-full text-center py-12 bg-white rounded-lg shadow-lg">
            <p class="text-gray-500 text-lg">אין עדיין נותני שירות במערכת</p>
            <p class="text-gray-400 mt-2">לחץ על "הוסף נותן שירות" כדי להתחיל</p>
          </div>
        `;
        return;
      }

      container.innerHTML = filtered.map(provider => `
        <div class="bg-white rounded-lg shadow-lg p-6 hover:shadow-xl transition-shadow">
          <div class="flex justify-between items-start mb-4">
            <div>
              <h3 class="text-xl font-bold text-gray-800">${provider.name || ''}</h3>
              <p class="text-blue-600 font-medium">${provider.profession || ''}</p>
            </div>
            ${isAdmin ? `
              <button onclick="deleteProvider('${provider.id}')" class="text-red-600 hover:bg-red-50 p-2 rounded-lg transition-colors" title="מחק">
                🗑️
              </button>
            ` : ''}
          </div>
          <div class="mb-4 text-2xl">
            ${'<span class="text-yellow-400">★</span>'.repeat(provider.rating || 0)}${'<span class="text-gray-300">★</span>'.repeat(5 - (provider.rating || 0))}
          </div>
          <div class="space-y-2 text-sm text-gray-600 mb-4">
            ${provider.phone ? `
              <div class="flex items-center gap-2">
                📞 <a href="tel:${provider.phone}" class="hover:text-blue-600">${provider.phone}</a>
              </div>
            ` : ''}
            ${provider.recommender ? `
              <div class="flex items-center gap-2">
                <span class="font-medium">ממליץ:</span>
                <span>${provider.recommender}</span>
              </div>
            ` : ''}
          </div>
          ${provider.notes ? `
            <div class="bg-gray-50 rounded-lg p-3 text-sm text-gray-700">
              <p class="font-medium mb-1">הערות:</p>
              <p>${provider.notes}</p>
            </div>
          ` : ''}
          <div class="mt-3 text-xs text-gray-400">
            נוסף ב: ${provider.createdAt ? new Date(provider.createdAt).toLocaleDateString('he-IL') : ''}
          </div>
        </div>
      `).join('');
    }

    // הוספת נותן שירות
    async function addProvider() {
      const name = document.getElementById('name').value.trim();
      const profession = document.getElementById('profession').value.trim();
      const phone = document.getElementById('phone').value.trim();
      const recommender = document.getElementById('recommender').value.trim();
      const rating = parseInt(document.getElementById('rating').value) || 0;
      const notes = document.getElementById('notes').value.trim();

      if (!name || !profession) {
        alert('נא למלא שם ומקצוע');
        return;
      }

      try {
        await addDoc(collection(db, "providers"), {
          name,
          profession,
          phone,
          recommender,
          rating,
          notes,
          createdAt: new Date().toISOString()
        });
        alert('נותן השירות נוסף בהצלחה!');
        document.getElementById('addForm').classList.add('hidden');
        clearForm();
      } catch (error) {
        console.error("שגיאה בהוספה:", error);
        alert('שגיאה בשמירת הנתונים: ' + error.message);
      }
    }

    // מחיקה
    window.deleteProvider = async function(id) {
      if (!isAdmin) {
        alert('יש להיכנס למצב מנהל כדי למחוק');
        return;
      }
      
      if (confirm('האם אתה בטוח שברצונך למחוק נותן שירות זה?')) {
        try {
          await deleteDoc(doc(db, "providers", id));
          alert('נותן השירות נמחק בהצלחה!');
        } catch (error) {
          console.error("שגיאה במחיקה:", error);
          alert('שגיאה במחיקת הנתונים: ' + error.message);
        }
      }
    };

    function clearForm() {
      document.getElementById('name').value = '';
      document.getElementById('profession').value = '';
      document.getElementById('phone').value = '';
      document.getElementById('recommender').value = '';
      document.getElementById('rating').value = '0';
      document.getElementById('notes').value = '';
      renderStars(0, true);
    }

    // Event Listeners
    document.getElementById('addButton').onclick = () => {
      document.getElementById('addForm').classList.toggle('hidden');
      if (!document.getElementById('addForm').classList.contains('hidden')) {
        renderStars(0, true);
      }
    };

    document.getElementById('saveButton').onclick = addProvider;
    document.getElementById('cancelButton').onclick = () => {
      document.getElementById('addForm').classList.add('hidden');
    };

    document.getElementById('searchInput').oninput = (e) => {
      searchTerm = e.target.value;
      renderProviders();
    };

    document.getElementById('adminToggle').onclick = () => {
      if (isAdmin) {
        isAdmin = false;
        document.getElementById('adminToggle').textContent = 'מצב מנהל';
        renderProviders();
      } else {
        document.getElementById('passwordModal').classList.remove('hidden');
      }
    };

    document.getElementById('passwordSubmit').onclick = () => {
      const password = document.getElementById('adminPassword').value;
      if (password === '6969') {
        isAdmin = true;
        document.getElementById('adminToggle').textContent = 'מצב מנהל ✓';
        document.getElementById('passwordModal').classList.add('hidden');
        document.getElementById('adminPassword').value = '';
        renderProviders();
      } else {
        alert('סיסמה שגויה!');
        document.getElementById('adminPassword').value = '';
      }
    };

    document.getElementById('passwordCancel').onclick = () => {
      document.getElementById('passwordModal').classList.add('hidden');
      document.getElementById('adminPassword').value = '';
    };

    document.getElementById('adminPassword').onkeypress = (e) => {
      if (e.key === 'Enter') {
        document.getElementById('passwordSubmit').click();
      }
    };

    // אתחול כוכבים
    renderStars(0, true);
  </script>
</body>
</html>
