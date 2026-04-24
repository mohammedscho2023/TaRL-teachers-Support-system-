<!DOCTYPE html>
<meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate" />
<meta http-equiv="Pragma" content="no-cache" />
<meta http-equiv="Expires" content="0" />
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
    <title>TaRL Teacher Support System | SMILE | Data Analysis & Reports</title>
    <!-- Tailwind CSS + Font Awesome + Chart.js -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
    <style>
        .modal { transition: opacity 0.2s ease; }
        .tab-btn { transition: all 0.2s; }
    </style>
</head>
<body class="bg-gray-50 font-sans">
    <div class="max-w-7xl mx-auto px-4 py-6">
        <!-- Header -->
        <div class="flex flex-wrap justify-between items-center mb-6 border-b pb-4">
            <div>
                <h1 class="text-2xl font-bold text-green-800"><i class="fas fa-chalkboard-user mr-2"></i>TaRL Teacher Support System</h1>
                <p class="text-sm text-gray-500">SMILE Project | Planning → Grade Management → Data Analysis & Narrative Reports</p>
            </div>
            <div class="flex items-center space-x-3">
                <span id="syncStatus" class="text-sm bg-white px-3 py-1 rounded-full shadow"><i class="fas fa-sync-alt mr-1"></i> Connecting...</span>
            </div>
        </div>

        <!-- Dashboard Cards -->
        <div class="grid grid-cols-1 sm:grid-cols-4 gap-4 mb-6">
            <div class="bg-white p-4 rounded-xl shadow-sm border-l-4 border-blue-500"><div class="text-gray-500 text-sm">Total Students</div><div class="text-3xl font-bold" id="statStudents">0</div></div>
            <div class="bg-white p-4 rounded-xl shadow-sm border-l-4 border-green-500"><div class="text-gray-500 text-sm">Reading Goal (Word+)</div><div class="text-3xl font-bold" id="statReadingGoal">0%</div></div>
            <div class="bg-white p-4 rounded-xl shadow-sm border-l-4 border-purple-500"><div class="text-gray-500 text-sm">Lesson Plans</div><div class="text-3xl font-bold" id="statLessons">0</div></div>
            <div class="bg-white p-4 rounded-xl shadow-sm border-l-4 border-orange-500"><div class="text-gray-500 text-sm">Avg Reading Score</div><div class="text-3xl font-bold" id="statAvgReading">--</div></div>
        </div>

        <!-- Tab Navigation -->
        <div class="flex flex-wrap border-b gap-2 mb-4">
            <button class="tab-btn py-2 px-4 font-medium text-gray-600 hover:text-blue-600 border-b-2 border-transparent" data-tab="dashboard">📊 Dashboard</button>
            <button class="tab-btn py-2 px-4 font-medium text-gray-600 hover:text-blue-600 border-b-2 border-transparent" data-tab="students">👩‍🎓 Students & Levels</button>
            <button class="tab-btn py-2 px-4 font-medium text-gray-600 hover:text-blue-600 border-b-2 border-transparent" data-tab="planning">📅 Lesson Planning</button>
            <button class="tab-btn py-2 px-4 font-medium text-gray-600 hover:text-blue-600 border-b-2 border-transparent" data-tab="gradebook">📊 Gradebook</button>
            <button class="tab-btn py-2 px-4 font-medium text-gray-600 hover:text-blue-600 border-b-2 border-transparent" data-tab="reports">📈 Reports & Analytics</button>
        </div>

        <!-- Tab: Dashboard (quick view) -->
        <div id="tab-dashboard" class="tab-content">
            <div class="bg-white rounded-xl shadow p-5">
                <h2 class="text-xl font-bold mb-3"><i class="fas fa-chart-line text-green-600"></i> Recent Activity & Live Sync</h2>
                <ul id="recentActivities" class="space-y-2 text-gray-700 max-h-64 overflow-y-auto"></ul>
                <div class="mt-5 p-3 bg-blue-50 rounded-lg text-sm"><i class="fas fa-cloud-upload-alt text-blue-600 mr-2"></i> All data is stored in Firebase Firestore. Edits on any device appear instantly.</div>
            </div>
        </div>

        <!-- Tab: Students -->
        <div id="tab-students" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow p-5">
                <div class="flex flex-wrap justify-between items-center mb-4"><h2 class="text-xl font-bold"><i class="fas fa-users"></i> Student Roster (TaRL Levels)</h2><button id="newStudentBtn" class="bg-green-600 text-white px-4 py-2 rounded-lg text-sm hover:bg-green-700"><i class="fas fa-plus"></i> Add Student</button></div>
                <div class="overflow-x-auto"><table class="min-w-full border text-sm"><thead class="bg-gray-100"><tr><th class="p-2 text-left">Name</th><th>Grade</th><th>Reading Level</th><th>Numeracy Level</th><th>Actions</th></tr></thead><tbody id="studentTableBody"></tbody></table></div>
                <div class="mt-3 text-xs text-gray-500">TaRL: Beginner → Letter → Word → Paragraph. Regroup every 4–6 weeks.</div>
            </div>
        </div>

        <!-- Tab: Lesson Planning -->
        <div id="tab-planning" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow p-5">
                <div class="flex justify-between mb-4"><h2 class="text-xl font-bold"><i class="fas fa-calendar-alt"></i> Weekly TaRL Lesson Plans</h2><button id="newLessonBtn" class="bg-blue-600 text-white px-4 py-2 rounded-lg text-sm"><i class="fas fa-plus"></i> Add Lesson</button></div>
                <div class="overflow-x-auto"><table class="min-w-full border text-sm"><thead class="bg-gray-100"><tr><th>Week</th><th>Day</th><th>Level</th><th>Objective</th><th>Activities</th><th>Actions</th></tr></thead><tbody id="lessonTableBody"></tbody></table></div>
            </div>
        </div>

        <!-- Tab: Gradebook -->
        <div id="tab-gradebook" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow p-5">
                <div class="flex justify-between mb-4"><h2 class="text-xl font-bold"><i class="fas fa-book-open"></i> Assessment & Grade Records</h2><button id="newGradeBtn" class="bg-purple-600 text-white px-4 py-2 rounded-lg text-sm"><i class="fas fa-plus"></i> Add Grade/Assessment</button></div>
                <div class="overflow-x-auto"><table class="min-w-full border text-sm"><thead class="bg-gray-100"><tr><th>Student</th><th>Assessment</th><th>Reading (%)</th><th>Numeracy (%)</th><th>Date</th><th>Actions</th></tr></thead><tbody id="gradeTableBody"></tbody></table></div>
            </div>
        </div>

        <!-- Tab: REPORTS & ANALYTICS (Narrative + Quantitative) -->
        <div id="tab-reports" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow p-5">
                <div class="flex flex-wrap justify-between items-center mb-4">
                    <h2 class="text-xl font-bold"><i class="fas fa-chart-pie"></i> Quantitative & Narrative Report</h2>
                    <button id="copyReportBtn" class="bg-indigo-600 text-white px-4 py-2 rounded-lg text-sm hover:bg-indigo-700"><i class="fas fa-copy"></i> Copy Full Report</button>
                </div>

                <!-- Key metrics row -->
                <div class="grid grid-cols-2 md:grid-cols-4 gap-3 mb-6 text-center">
                    <div class="bg-gray-100 p-2 rounded"><span class="font-bold">Total Students</span><br><span id="reportTotalStudents" class="text-2xl">0</span></div>
                    <div class="bg-gray-100 p-2 rounded"><span class="font-bold">Reading Proficient (Word+)</span><br><span id="reportReadingProficient" class="text-2xl">0%</span></div>
                    <div class="bg-gray-100 p-2 rounded"><span class="font-bold">Avg Reading Score</span><br><span id="reportAvgReadingScore" class="text-2xl">--</span></div>
                    <div class="bg-gray-100 p-2 rounded"><span class="font-bold">Avg Numeracy Score</span><br><span id="reportAvgNumeracyScore" class="text-2xl">--</span></div>
                </div>

                <!-- Charts row -->
                <div class="grid md:grid-cols-2 gap-6 mb-6">
                    <div><canvas id="readingLevelChart" width="400" height="250"></canvas></div>
                    <div><canvas id="numeracyLevelChart" width="400" height="250"></canvas></div>
                </div>

                <!-- Assessment trend line -->
                <div class="mb-6"><canvas id="trendChart" width="800" height="200"></canvas></div>

                <!-- Narrative report (auto-generated) -->
                <div class="bg-gray-50 p-4 rounded-lg border border-gray-200">
                    <h3 class="font-bold text-lg mb-2"><i class="fas fa-file-alt"></i> Narrative Report (Auto‑generated)</h3>
                    <div id="narrativeText" class="text-gray-700 whitespace-pre-line"></div>
                </div>
                <p class="text-xs text-gray-400 mt-4">*Based on real‑time data from Firebase. Update students or assessments to refresh the report.</p>
            </div>
        </div>

        <!-- MODALS (same as before) -->
        <div id="studentModal" class="fixed inset-0 bg-gray-900 bg-opacity-50 flex items-center justify-center hidden z-50"><div class="bg-white rounded-lg w-full max-w-md p-6"><div class="flex justify-between"><h3 id="studentModalTitle" class="text-lg font-bold">Add Student</h3><button class="close-modal text-gray-500 text-2xl">&times;</button></div><form id="studentForm"><input type="hidden" id="studentId"><div class="mt-2"><label class="block font-medium">Full Name</label><input type="text" id="studentName" required class="w-full border p-2 rounded"></div><div class="mt-2"><label>Grade</label><select id="studentGrade" class="w-full border p-2 rounded"><option>3</option><option>4</option><option>5</option></select></div><div class="mt-2"><label>Reading Level (TaRL)</label><select id="readingLevel" class="w-full border p-2 rounded"><option>Beginner</option><option>Letter</option><option>Word</option><option>Paragraph</option></select></div><div class="mt-2"><label>Numeracy Level</label><select id="numeracyLevel" class="w-full border p-2 rounded"><option>Beginner</option><option>Number ID</option><option>Addition</option><option>Subtraction</option></select></div><div class="mt-4 flex justify-end"><button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded">Save Student</button></div></form></div></div>

        <div id="lessonModal" class="fixed inset-0 bg-gray-900 bg-opacity-50 flex items-center justify-center hidden z-50"><div class="bg-white rounded-lg w-full max-w-md p-6"><div class="flex justify-between"><h3 id="lessonModalTitle" class="text-lg font-bold">Lesson Plan</h3><button class="close-modal text-gray-500 text-2xl">&times;</button></div><form id="lessonForm"><input type="hidden" id="lessonId"><div><label>Week</label><input type="text" id="lessonWeek" class="w-full border p-2 rounded"></div><div><label>Day</label><input type="text" id="lessonDay" class="w-full border p-2 rounded"></div><div><label>TaRL Level</label><select id="lessonLevel" class="w-full border p-2 rounded"><option>Beginner</option><option>Letter</option><option>Word</option><option>Paragraph</option></select></div><div><label>Learning Objective</label><textarea id="lessonObjective" rows="2" class="w-full border p-2 rounded"></textarea></div><div><label>Activities / Materials</label><textarea id="lessonActivities" rows="2" class="w-full border p-2 rounded"></textarea></div><div class="mt-4 flex justify-end"><button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded">Save Lesson</button></div></form></div></div>

        <div id="gradeModal" class="fixed inset-0 bg-gray-900 bg-opacity-50 flex items-center justify-center hidden z-50"><div class="bg-white rounded-lg w-full max-w-md p-6"><div class="flex justify-between"><h3 id="gradeModalTitle" class="text-lg font-bold">Assessment / Grade</h3><button class="close-modal text-gray-500 text-2xl">&times;</button></div><form id="gradeForm"><input type="hidden" id="gradeId"><div><label>Student</label><select id="gradeStudentId" required class="w-full border p-2 rounded"></select></div><div><label>Assessment Type</label><select id="gradeType" class="w-full border p-2 rounded"><option>Baseline</option><option>Mid‑term</option><option>Endline</option><option>Weekly Formative</option></select></div><div><label>Reading Score (%)</label><input type="number" id="gradeReading" class="w-full border p-2 rounded"></div><div><label>Numeracy Score (%)</label><input type="number" id="gradeNumeracy" class="w-full border p-2 rounded"></div><div><label>Date</label><input type="date" id="gradeDate" class="w-full border p-2 rounded"></div><div class="mt-4 flex justify-end"><button type="submit" class="bg-purple-600 text-white px-4 py-2 rounded">Save Grade</button></div></form></div></div>

        <div class="text-center text-xs text-gray-400 mt-8 border-t pt-4">TaRL Teacher Support System – Real‑time sync + Automated Data Analysis & Narrative Reports</div>
    </div>

    <script type="module">
        // ======================= FIREBASE CONFIGURATION =======================
        // REPLACE WITH YOUR OWN FIREBASE PROJECT CREDENTIALS
        const firebaseConfig = {
            apiKey: "AIzaSyDummyReplaceWithYourActualKey",
            authDomain: "your-project.firebaseapp.com",
            projectId: "your-project-id",
            storageBucket: "your-project.appspot.com",
            messagingSenderId: "123456789012",
            appId: "1:123456789012:web:abc123def456"
        };

        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
        import { getFirestore, collection, doc, onSnapshot, addDoc, updateDoc, deleteDoc, query, orderBy } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";

        let db;
        let initError = false;
        try {
            const app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            console.log("Firebase ready");
        } catch(e) { initError = true; console.error(e); alert("Firebase config error – please check credentials"); }

        const studentsCol = db ? collection(db, "students") : null;
        const lessonsCol = db ? collection(db, "lessonPlans") : null;
        const gradesCol = db ? collection(db, "grades") : null;

        let students = [], lessons = [], grades = [];
        let readingChart, numeracyChart, trendChart;

        // DOM elements
        const syncStatusSpan = document.getElementById("syncStatus");
        const studentTableBody = document.getElementById("studentTableBody");
        const lessonTableBody = document.getElementById("lessonTableBody");
        const gradeTableBody = document.getElementById("gradeTableBody");

        function updateSyncStatus(connected) {
            if(syncStatusSpan) {
                if(connected) { syncStatusSpan.innerHTML = '<i class="fas fa-check-circle text-green-600"></i> Live sync • cross-device'; }
                else { syncStatusSpan.innerHTML = '<i class="fas fa-exclamation-triangle text-yellow-600"></i> Offline mode'; }
            }
        }

        function escapeHtml(str) { if(!str) return ''; return str.replace(/[&<>]/g, m => ({ '&':'&amp;', '<':'&lt;', '>':'&gt;' }[m])); }
        function getLevelBadge(lvl) { const map = { Beginner:"bg-gray-300", Letter:"bg-blue-100", Word:"bg-green-100", Paragraph:"bg-purple-100" }; return map[lvl]||"bg-gray-200"; }

        // Render functions (students, lessons, grades) – same as before
        function renderStudents() {
            if(!studentTableBody) return;
            studentTableBody.innerHTML = students.map(s => `<tr class="border-b"><td class="p-2 font-medium">${escapeHtml(s.name)}</td><td class="p-2">Grade ${s.grade||'3'}</td><td class="p-2"><span class="px-2 py-1 rounded-full text-xs font-bold ${getLevelBadge(s.readingLevel)}">${s.readingLevel||'Beginner'}</span></td><td class="p-2"><span class="px-2 py-1 rounded-full text-xs font-bold ${getLevelBadge(s.numeracyLevel)}">${s.numeracyLevel||'Beginner'}</span></td><td class="p-2"><button class="edit-student text-blue-600 mr-2" data-id="${s.id}"><i class="fas fa-edit"></i></button><button class="delete-student text-red-600" data-id="${s.id}"><i class="fas fa-trash"></i></button></td></tr>`).join('');
            document.querySelectorAll(".edit-student").forEach(b => b.addEventListener("click", () => editStudent(b.dataset.id)));
            document.querySelectorAll(".delete-student").forEach(b => b.addEventListener("click", () => deleteStudent(b.dataset.id)));
        }
        function renderLessons() {
            if(!lessonTableBody) return;
            lessonTableBody.innerHTML = lessons.map(l => `<tr class="border-b"><td class="p-2">Week ${l.week||'-'}</td><td class="p-2">${l.day||'-'}</td><td class="p-2">${l.levelGroup||'-'}</td><td class="p-2 max-w-xs">${escapeHtml(l.objective||'')}</td><td class="p-2 max-w-xs">${escapeHtml(l.activities||'')}</td><td class="p-2"><button class="edit-lesson text-blue-600 mr-2" data-id="${l.id}"><i class="fas fa-edit"></i></button><button class="delete-lesson text-red-600" data-id="${l.id}"><i class="fas fa-trash"></i></button></td></tr>`).join('');
            document.querySelectorAll(".edit-lesson").forEach(b => b.addEventListener("click", () => editLesson(b.dataset.id)));
            document.querySelectorAll(".delete-lesson").forEach(b => b.addEventListener("click", () => deleteLesson(b.dataset.id)));
        }
        function renderGrades() {
            if(!gradeTableBody) return;
            gradeTableBody.innerHTML = grades.map(g => {
                const stu = students.find(s => s.id === g.studentId);
                return `<tr class="border-b"><td class="p-2">${escapeHtml(stu?.name||'Unknown')}</td><td class="p-2">${g.assessmentType||'Formative'}</td><td class="p-2">${g.readingScore??'-'}</td><td class="p-2">${g.numeracyScore??'-'}</td><td class="p-2">${g.date||''}</td><td class="p-2"><button class="edit-grade text-blue-600 mr-2" data-id="${g.id}"><i class="fas fa-edit"></i></button><button class="delete-grade text-red-600" data-id="${g.id}"><i class="fas fa-trash"></i></button></td></tr>`;
            }).join('');
            document.querySelectorAll(".edit-grade").forEach(b => b.addEventListener("click", () => editGrade(b.dataset.id)));
            document.querySelectorAll(".delete-grade").forEach(b => b.addEventListener("click", () => deleteGrade(b.dataset.id)));
        }

        // CRUD (same as before)
        async function addStudent(data) { if(db) await addDoc(studentsCol, data); }
        async function updateStudent(id, data) { if(db) await updateDoc(doc(db, "students", id), data); }
        async function deleteStudent(id) { if(confirm("Delete student?")) if(db) await deleteDoc(doc(db, "students", id)); }
        async function addLesson(data) { if(db) await addDoc(lessonsCol, data); }
        async function updateLesson(id, data) { if(db) await updateDoc(doc(db, "lessonPlans", id), data); }
        async function deleteLesson(id) { if(confirm("Delete lesson?")) if(db) await deleteDoc(doc(db, "lessonPlans", id)); }
        async function addGrade(data) { if(db) await addDoc(gradesCol, data); }
        async function updateGrade(id, data) { if(db) await updateDoc(doc(db, "grades", id), data); }
        async function deleteGrade(id) { if(confirm("Delete grade?")) if(db) await deleteDoc(doc(db, "grades", id)); }

        function editStudent(id) { const s=students.find(x=>x.id===id); if(s){ document.getElementById("studentId").value=s.id; document.getElementById("studentName").value=s.name; document.getElementById("studentGrade").value=s.grade; document.getElementById("readingLevel").value=s.readingLevel; document.getElementById("numeracyLevel").value=s.numeracyLevel; document.getElementById("studentModalTitle").innerText="Edit Student"; document.getElementById("studentModal").classList.remove("hidden"); } }
        function editLesson(id) { const l=lessons.find(x=>x.id===id); if(l){ document.getElementById("lessonId").value=l.id; document.getElementById("lessonWeek").value=l.week; document.getElementById("lessonDay").value=l.day; document.getElementById("lessonLevel").value=l.levelGroup; document.getElementById("lessonObjective").value=l.objective; document.getElementById("lessonActivities").value=l.activities; document.getElementById("lessonModalTitle").innerText="Edit Lesson"; document.getElementById("lessonModal").classList.remove("hidden"); } }
        function editGrade(id) { const g=grades.find(x=>x.id===id); if(g){ document.getElementById("gradeId").value=g.id; document.getElementById("gradeStudentId").value=g.studentId; document.getElementById("gradeType").value=g.assessmentType; document.getElementById("gradeReading").value=g.readingScore; document.getElementById("gradeNumeracy").value=g.numeracyScore; document.getElementById("gradeDate").value=g.date; document.getElementById("gradeModalTitle").innerText="Edit Grade"; document.getElementById("gradeModal").classList.remove("hidden"); } }

        // ---------- REPORTING & ANALYTICS ----------
        function updateReportsAndCharts() {
            // update metrics on reports tab
            const total = students.length;
            const proficient = students.filter(s => s.readingLevel === "Word" || s.readingLevel === "Paragraph").length;
            const pctProficient = total ? Math.round((proficient/total)*100) : 0;
            document.getElementById("reportTotalStudents").innerText = total;
            document.getElementById("reportReadingProficient").innerText = pctProficient+"%";
            // average scores from latest assessment per student? we compute all grade entries average
            let readingScores = [], numeracyScores = [];
            grades.forEach(g => { if(g.readingScore != null) readingScores.push(Number(g.readingScore)); if(g.numeracyScore != null) numeracyScores.push(Number(g.numeracyScore)); });
            const avgReading = readingScores.length ? Math.round(readingScores.reduce((a,b)=>a+b,0)/readingScores.length) : null;
            const avgNumeracy = numeracyScores.length ? Math.round(numeracyScores.reduce((a,b)=>a+b,0)/numeracyScores.length) : null;
            document.getElementById("reportAvgReadingScore").innerHTML = avgReading !== null ? avgReading : "--";
            document.getElementById("reportAvgNumeracyScore").innerHTML = avgNumeracy !== null ? avgNumeracy : "--";
            document.getElementById("statAvgReading").innerHTML = avgReading !== null ? avgReading+"%" : "--%";

            // update dashboard cards
            document.getElementById("statStudents").innerText = total;
            document.getElementById("statReadingGoal").innerText = pctProficient+"%";
            document.getElementById("statLessons").innerText = lessons.length;

            // reading level distribution
            const readingCounts = { Beginner:0, Letter:0, Word:0, Paragraph:0 };
            students.forEach(s => { if(s.readingLevel) readingCounts[s.readingLevel]++; });
            if(readingChart) readingChart.destroy();
            const ctx1 = document.getElementById("readingLevelChart").getContext("2d");
            readingChart = new Chart(ctx1, { type:"bar", data:{ labels:["Beginner","Letter","Word","Paragraph"], datasets:[{ label:"Students", data:[readingCounts.Beginner,readingCounts.Letter,readingCounts.Word,readingCounts.Paragraph], backgroundColor:"#3b82f6" }] }, options:{ responsive:true, maintainAspectRatio:true } });
            
            // numeracy levels (simplified)
            const numeracyCounts = { Beginner:0, "Number ID":0, Addition:0, Subtraction:0 };
            students.forEach(s => { let lvl = s.numeracyLevel || "Beginner"; if(numeracyCounts[lvl] !== undefined) numeracyCounts[lvl]++; else numeracyCounts.Beginner++; });
            if(numeracyChart) numeracyChart.destroy();
            const ctx2 = document.getElementById("numeracyLevelChart").getContext("2d");
            numeracyChart = new Chart(ctx2, { type:"bar", data:{ labels:["Beginner","Number ID","Addition","Subtraction"], datasets:[{ label:"Students", data:[numeracyCounts.Beginner, numeracyCounts["Number ID"], numeracyCounts.Addition, numeracyCounts.Subtraction], backgroundColor:"#10b981" }] }, options:{ responsive:true } });

            // trend chart: average reading score per assessment type (Baseline, Mid-term, Endline)
            const trendMap = { "Baseline":[], "Mid‑term":[], "Endline":[] };
            grades.forEach(g => { if(g.readingScore && trendMap[g.assessmentType]) trendMap[g.assessmentType].push(Number(g.readingScore)); });
            const trendLabels = ["Baseline", "Mid‑term", "Endline"];
            const trendData = trendLabels.map(type => { let arr = trendMap[type]; return arr.length ? Math.round(arr.reduce((a,b)=>a+b,0)/arr.length) : null; });
            if(trendChart) trendChart.destroy();
            const ctx3 = document.getElementById("trendChart").getContext("2d");
            trendChart = new Chart(ctx3, { type:"line", data:{ labels:trendLabels, datasets:[{ label:"Average Reading Score (%)", data:trendData, borderColor:"#f59e0b", tension:0.2, fill:false }] }, options:{ responsive:true, scales:{ y:{ min:0, max:100 } } } });

            // ---- generate narrative ----
            let narrative = `📋 QUANTITATIVE SUMMARY (based on ${total} students)\n`;
            narrative += `• Reading proficiency (Word/Paragraph): ${pctProficient}% (${proficient} out of ${total})\n`;
            narrative += `• Average reading assessment score: ${avgReading !== null ? avgReading+"%" : "Not enough data"}\n`;
            narrative += `• Average numeracy assessment score: ${avgNumeracy !== null ? avgNumeracy+"%" : "Not enough data"}\n`;
            narrative += `• Total lesson plans created: ${lessons.length}\n\n`;
            narrative += `📊 LEVEL DISTRIBUTION\n`;
            narrative += `Reading: Beginner ${readingCounts.Beginner}, Letter ${readingCounts.Letter}, Word ${readingCounts.Word}, Paragraph ${readingCounts.Paragraph}\n`;
            narrative += `Numeracy: Beginner ${numeracyCounts.Beginner}, Number ID ${numeracyCounts["Number ID"]}, Addition ${numeracyCounts.Addition}, Subtraction ${numeracyCounts.Subtraction}\n\n`;
            narrative += `📈 INTERPRETATION & RECOMMENDATIONS\n`;
            if(pctProficient >= 80) narrative += `✅ Excellent: Most students have reached Word level or above. Focus on fluency and comprehension for advanced groups.\n`;
            else if(pctProficient >= 50) narrative += `⚠️ Good progress: Over half are at Word level. Continue intensive TaRL for Letter/Beginner groups.\n`;
            else narrative += `⚠️ Critical need: Majority are still at Beginner/Letter. Prioritize letter recognition and phonics. Use peer coaching.\n`;
            if(avgReading !== null && avgReading < 50) narrative += `🔴 Low average reading scores. Reassess grouping and increase reading club frequency.\n`;
            if(lessons.length < total) narrative += `📌 Consider creating more TaRL lesson plans (one per level per week) to support differentiation.\n`;
            narrative += `\n🔄 Next steps: Regroup students every 4 weeks. Use assessment data to move children to higher levels. Celebrate small wins.`;
            document.getElementById("narrativeText").innerText = narrative;
        }

        // Copy report to clipboard
        document.getElementById("copyReportBtn")?.addEventListener("click", () => {
            const narrative = document.getElementById("narrativeText")?.innerText || "";
            const extra = `\n\n--- TaRL Teacher Support System Report ---\nGenerated: ${new Date().toLocaleString()}`;
            navigator.clipboard.writeText(narrative + extra).then(() => alert("Full report copied to clipboard!"));
        });

        // Real-time subscriptions
        function subscribeFirestore() {
            if(!db) { updateSyncStatus(false); return; }
            updateSyncStatus(true);
            onSnapshot(query(studentsCol, orderBy("name")), snap => { students = snap.docs.map(d=>d.data()); students.forEach((d,i)=>{ students[i].id=snap.docs[i].id; }); renderStudents(); renderGrades(); updateReportsAndCharts(); updateDashboardRecent(); });
            onSnapshot(query(lessonsCol, orderBy("week")), snap => { lessons = snap.docs.map(d=>d.data()); lessons.forEach((d,i)=>{ lessons[i].id=snap.docs[i].id; }); renderLessons(); updateReportsAndCharts(); });
            onSnapshot(gradesCol, snap => { grades = snap.docs.map(d=>d.data()); grades.forEach((d,i)=>{ grades[i].id=snap.docs[i].id; }); renderGrades(); updateReportsAndCharts(); });
        }
        function updateDashboardRecent() {
            const recentList = document.getElementById("recentActivities");
            if(recentList) recentList.innerHTML = grades.slice(-5).reverse().map(g => { const s=students.find(x=>x.id===g.studentId); return `<li class="text-sm">${s?.name||'?'} – ${g.assessmentType}: R ${g.readingScore??'N/A'} / N ${g.numeracyScore??'N/A'}</li>`; }).join('');
        }

        // Forms and modals init
        function setupForms() {
            document.getElementById("studentForm")?.addEventListener("submit", async(e)=>{ e.preventDefault(); const id=document.getElementById("studentId").value; const data={ name:document.getElementById("studentName").value, grade:document.getElementById("studentGrade").value, readingLevel:document.getElementById("readingLevel").value, numeracyLevel:document.getElementById("numeracyLevel").value }; if(id) await updateStudent(id,data); else await addStudent(data); document.getElementById("studentModal").classList.add("hidden"); document.getElementById("studentForm").reset(); document.getElementById("studentId").value=""; });
            document.getElementById("lessonForm")?.addEventListener("submit", async(e)=>{ e.preventDefault(); const id=document.getElementById("lessonId").value; const data={ week:document.getElementById("lessonWeek").value, day:document.getElementById("lessonDay").value, levelGroup:document.getElementById("lessonLevel").value, objective:document.getElementById("lessonObjective").value, activities:document.getElementById("lessonActivities").value }; if(id) await updateLesson(id,data); else await addLesson(data); document.getElementById("lessonModal").classList.add("hidden"); document.getElementById("lessonForm").reset(); document.getElementById("lessonId").value=""; });
            document.getElementById("gradeForm")?.addEventListener("submit", async(e)=>{ e.preventDefault(); const id=document.getElementById("gradeId").value; const data={ studentId:document.getElementById("gradeStudentId").value, assessmentType:document.getElementById("gradeType").value, readingScore:parseInt(document.getElementById("gradeReading").value)||null, numeracyScore:parseInt(document.getElementById("gradeNumeracy").value)||null, date:document.getElementById("gradeDate").value }; if(id) await updateGrade(id,data); else await addGrade(data); document.getElementById("gradeModal").classList.add("hidden"); document.getElementById("gradeForm").reset(); document.getElementById("gradeId").value=""; });
            document.querySelectorAll(".close-modal").forEach(btn=>btn.addEventListener("click",()=>{ document.getElementById("studentModal")?.classList.add("hidden"); document.getElementById("lessonModal")?.classList.add("hidden"); document.getElementById("gradeModal")?.classList.add("hidden"); }));
            document.getElementById("newStudentBtn")?.addEventListener("click",()=>{ document.getElementById("studentForm").reset(); document.getElementById("studentId").value=""; document.getElementById("studentModalTitle").innerText="Add Student"; document.getElementById("studentModal").classList.remove("hidden"); });
            document.getElementById("newLessonBtn")?.addEventListener("click",()=>{ document.getElementById("lessonForm").reset(); document.getElementById("lessonId").value=""; document.getElementById("lessonModalTitle").innerText="Add Lesson Plan"; document.getElementById("lessonModal").classList.remove("hidden"); });
            document.getElementById("newGradeBtn")?.addEventListener("click",()=>{ document.getElementById("gradeForm").reset(); document.getElementById("gradeId").value=""; document.getElementById("gradeModalTitle").innerText="Add Assessment"; document.getElementById("gradeModal").classList.remove("hidden"); });
            setInterval(()=>{ const sel=document.getElementById("gradeStudentId"); if(sel){ const cur=sel.value; sel.innerHTML='<option value="">-- Select Student --</option>'+students.map(s=>`<option value="${s.id}">${escapeHtml(s.name)} (G${s.grade})</option>`).join(''); if(cur && students.find(s=>s.id===cur)) sel.value=cur; } }, 800);
        }

        function initTabs() {
            const tabs = document.querySelectorAll(".tab-btn"), contents = document.querySelectorAll(".tab-content");
            tabs.forEach(btn => btn.addEventListener("click", () => {
                const target = btn.dataset.tab;
                tabs.forEach(b => b.classList.remove("border-blue-600","text-blue-600","border-b-2"));
                btn.classList.add("border-blue-600","text-blue-600","border-b-2");
                contents.forEach(c => c.classList.add("hidden"));
                document.getElementById(`tab-${target}`)?.classList.remove("hidden");
                if(target === "reports") setTimeout(()=>updateReportsAndCharts(), 100);
            }));
        }

        window.addEventListener("DOMContentLoaded", () => { setupForms(); initTabs(); subscribeFirestore(); });
    </script>
</body>
</html>
