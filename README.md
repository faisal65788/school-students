<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام إدارة أسماء الطلاب</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700&display=swap" rel="stylesheet">
    <style>body { font-family: 'Tajawal', sans-serif; }</style>
</head>
<body class="bg-slate-100 min-h-screen p-4 md:p-8">
    <div class="max-w-4xl mx-auto space-y-6">
        <header class="bg-indigo-700 text-white p-6 rounded-2xl shadow-lg text-center">
            <h1 class="text-3xl font-bold">نظام تسجيل وإدارة أسماء الطلاب</h1>
            <p class="text-indigo-200 mt-2">إضافة، عرض، وتصدير قائمة طلاب المدرسة</p>
        </header>

        <section class="bg-white p-6 rounded-2xl shadow-md">
            <h2 class="text-xl font-bold text-gray-800 mb-4 border-b pb-2">تسجيل طالب جديد</h2>
            <form id="studentForm" class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">اسم الطالب الكامل</label>
                    <input type="text" id="studentName" required placeholder="مثال: أحمد محمد علي" class="w-full p-3 border rounded-xl focus:ring-2 focus:ring-indigo-500 outline-none">
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">الصف / الفصل</label>
                    <input type="text" id="studentClass" required placeholder="مثال: الثالث ثانوي - أ" class="w-full p-3 border rounded-xl focus:ring-2 focus:ring-indigo-500 outline-none">
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">رقم التواصل (اختياري)</label>
                    <input type="tel" id="studentPhone" placeholder="05XXXXXXXX" class="w-full p-3 border rounded-xl focus:ring-2 focus:ring-indigo-500 outline-none">
                </div>
                <div class="md:col-span-3 text-left">
                    <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-3 rounded-xl font-bold transition shadow-md w-full md:w-auto">
                        + إضافة الطالب
                    </button>
                </div>
            </form>
        </section>

        <section class="bg-white p-6 rounded-2xl shadow-md space-y-4">
            <div class="flex flex-col md:flex-row justify-between items-center gap-4 border-b pb-4">
                <div>
                    <h2 class="text-xl font-bold text-gray-800">قائمة الطلاب المسجلين</h2>
                    <span id="studentCount" class="text-sm text-gray-500">إجمالي الطلاب: 0</span>
                </div>
                <div class="flex items-center gap-2 w-full md:w-auto">
                    <input type="text" id="searchBar" onkeyup="filterStudents()" placeholder="بحث باسم الطالب..." class="p-2 border rounded-lg text-sm w-full md:w-48">
                    <button onclick="exportCSV()" class="bg-emerald-600 hover:bg-emerald-700 text-white px-4 py-2 rounded-lg text-sm font-semibold whitespace-nowrap">
                        تصدير Excel/CSV
                    </button>
                </div>
            </div>

            <div class="overflow-x-auto">
                <table class="w-full text-right border-collapse">
                    <thead>
                        <tr class="bg-slate-50 text-gray-600 text-sm border-b">
                            <th class="p-3">#</th>
                            <th class="p-3">اسم الطالب</th>
                            <th class="p-3">الصف</th>
                            <th class="p-3">رقم الهاتف</th>
                            <th class="p-3 text-center">إجراءات</th>
                        </tr>
                    </thead>
                    <tbody id="studentTableBody" class="divide-y text-gray-700"></tbody>
                </table>
            </div>
        </section>
    </div>

    <script>
        let students = JSON.parse(localStorage.getItem('school_students')) || [];
        const studentForm = document.getElementById('studentForm');
        const studentTableBody = document.getElementById('studentTableBody');
        const studentCount = document.getElementById('studentCount');

        function renderStudents(list = students) {
            studentTableBody.innerHTML = '';
            studentCount.textContent = `إجمالي الطلاب: ${list.length}`;
            if (list.length === 0) {
                studentTableBody.innerHTML = `<tr><td colspan="5" class="text-center p-6 text-gray-400">لا يوجد طلاب مسجلون حتى الآن</td></tr>`;
                return;
            }
            list.forEach((student, index) => {
                const row = document.createElement('tr');
                row.className = 'hover:bg-slate-50 transition';
                row.innerHTML = `
                    <td class="p-3 font-semibold text-gray-400">${index + 1}</td>
                    <td class="p-3 font-medium">${student.name}</td>
                    <td class="p-3"><span class="bg-indigo-50 text-indigo-700 px-2 py-1 rounded-md text-sm">${student.class}</span></td>
                    <td class="p-3 text-sm text-gray-500" dir="ltr">${student.phone || '-'}</td>
                    <td class="p-3 text-center">
                        <button onclick="deleteStudent(${student.id})" class="text-red-500 hover:text-red-700 font-semibold text-sm">حذف</button>
                    </td>
                `;
                studentTableBody.appendChild(row);
            });
        }

        studentForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const newStudent = {
                id: Date.now(),
                name: document.getElementById('studentName').value.trim(),
                class: document.getElementById('studentClass').value.trim(),
                phone: document.getElementById('studentPhone').value.trim()
            };
            students.push(newStudent);
            localStorage.setItem('school_students', JSON.stringify(students));
            studentForm.reset();
            renderStudents();
        });

        function deleteStudent(id) {
            students = students.filter(s => s.id !== id);
            localStorage.setItem('school_students', JSON.stringify(students));
            renderStudents();
        }

        function filterStudents() {
            const query = document.getElementById('searchBar').value.toLowerCase();
            const filtered = students.filter(s => s.name.toLowerCase().includes(query) || s.class.toLowerCase().includes(query));
            renderStudents(filtered);
        }

        function exportCSV() {
            if (students.length === 0) return alert('لا توجد بيانات للتصدير!');
            let csvContent = "\uFEFFالرقم,اسم الطالب,الصف,رقم الهاتف\n";
            students.forEach((s, idx) => {
                csvContent += `${idx + 1},"${s.name}","${s.class}","${s.phone}"\n`;
            });
            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement("a");
            link.href = URL.createObjectURL(blob);
            link.download = "قائمة_الطلاب.csv";
            link.click();
        }
        renderStudents();
    </script>
</body>
</html>
