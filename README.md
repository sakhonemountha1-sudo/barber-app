<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Barber Pro POS - ระบบบันทึกงานช่างผม</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;600&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Kanit', sans-serif; }
        .glass-effect { background: rgba(255, 255, 255, 0.9); backdrop-filter: blur(10px); }
    </style>
</head>
<body class="bg-slate-100 pb-24">

    <nav class="bg-slate-900 text-white p-5 shadow-xl sticky top-0 z-50">
        <div class="max-w-md mx-auto flex justify-between items-center">
            <div>
                <h1 class="text-xl font-bold tracking-tight text-amber-400">BARBER PRO</h1>
                <p class="text-[10px] text-slate-400 uppercase tracking-widest">Management System</p>
            </div>
            <div class="text-right">
                <div id="totalToday" class="text-lg font-bold">฿0</div>
                <p class="text-[10px] text-slate-400">ยอดรวมวันนี้</p>
            </div>
        </div>
    </nav>

    <div class="p-4 max-w-md mx-auto">
        <div class="bg-white p-6 rounded-3xl shadow-sm mb-8 border border-slate-200">
            <h2 class="text-slate-800 font-bold mb-5 flex items-center text-lg">
                <i class="fas fa-plus-circle mr-2 text-amber-500"></i> เพิ่มรายการใหม่
            </h2>
            <form id="jobForm" class="space-y-4">
                <div>
                    <label class="block text-xs font-semibold text-slate-500 mb-1 ml-1">ชื่อลูกค้า</label>
                    <input type="text" id="custName" placeholder="ระบุชื่อลูกค้า..." required 
                        class="w-full p-4 bg-slate-50 border border-slate-200 rounded-2xl focus:ring-2 focus:ring-amber-500 focus:border-transparent outline-none transition-all">
                </div>
                
                <div class="grid grid-cols-2 gap-4">
                    <div>
                        <label class="block text-xs font-semibold text-slate-500 mb-1 ml-1">บริการ</label>
                        <select id="service" class="w-full p-4 bg-slate-50 border border-slate-200 rounded-2xl outline-none appearance-none font-medium">
                            <option value="300">ตัดผมชาย (300.-)</option>
                            <option value="150">สระไดร์ (150.-)</option>
                            <option value="1200">ทำสีผม (1,200.-)</option>
                            <option value="500">นวดหน้า (500.-)</option>
                            <option value="100">กันคิ้ว (100.-)</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-500 mb-1 ml-1">ช่างผู้ดูแล</label>
                        <select id="staffName" class="w-full p-4 bg-slate-50 border border-slate-200 rounded-2xl outline-none text-indigo-600 font-bold">
                            <option>ช่างเอ</option>
                            <option>ช่างบี</option>
                            <option>ช่างซี</option>
                        </select>
                    </div>
                </div>

                <button type="submit" id="submitBtn" 
                    class="w-full bg-slate-900 text-white py-4 rounded-2xl font-bold text-lg shadow-lg active:scale-95 transition-transform flex justify-center items-center">
                    <span>บันทึกรายการ</span>
                </button>
            </form>
        </div>

        <div class="space-y-4">
            <div class="flex justify-between items-end px-1">
                <h2 class="font-bold text-slate-800 text-lg">ประวัติงานวันนี้</h2>
                <button onclick="fetchHistory()" class="text-slate-500 text-sm hover:text-amber-600 transition">
                    <i class="fas fa-sync-alt mr-1"></i> อัปเดตข้อมูล
                </button>
            </div>
            
            <div id="historyList" class="space-y-3">
                <div class="text-center py-10">
                    <i class="fas fa-circle-notch animate-spin text-slate-300 text-3xl"></i>
                </div>
            </div>
        </div>
    </div>

    <div id="statusToast" class="fixed bottom-10 left-1/2 -translate-x-1/2 glass-effect px-6 py-3 rounded-full shadow-2xl border border-white hidden text-sm font-medium">
    </div>

    <script>
        // *** ใส่ URL จาก Google Apps Script ตรงนี้ ***
        const scriptURL = 'ใส่_URL_ที่ก๊อปมาตรงนี้ครับ'; 
        
        const form = document.getElementById('jobForm');
        const submitBtn = document.getElementById('submitBtn');

        // 1. ฟังก์ชันดึงประวัติ (GET)
        async function fetchHistory() {
            const list = document.getElementById('historyList');
            const totalDisplay = document.getElementById('totalToday');
            
            try {
                const response = await fetch(scriptURL);
                const data = await response.json();
                
                let total = 0;
                if (data.length === 0) {
                    list.innerHTML = `
                        <div class="text-center p-10 bg-white rounded-3xl border border-dashed border-slate-300 text-slate-400">
                            <i class="fas fa-calendar-day mb-2 text-2xl"></i>
                            <p>วันนี้ยังไม่มีรายการบันทึก</p>
                        </div>`;
                    totalDisplay.innerText = `฿0`;
                    return;
                }

                list.innerHTML = data.map(row => {
                    total += Number(row[3]);
                    const time = new Date(row[0]).toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' });
                    return `
                        <div class="bg-white p-4 rounded-2xl flex justify-between items-center shadow-sm border border-slate-100 hover:border-amber-200 transition-colors">
                            <div class="flex items-center">
                                <div class="w-10 h-10 bg-slate-100 rounded-full flex items-center justify-center mr-3">
                                    <i class="fas fa-user text-slate-400 text-sm"></i>
                                </div>
                                <div>
                                    <p class="font-bold text-slate-800">${row[1]}</p>
                                    <p class="text-[10px] text-slate-400 font-medium uppercase tracking-tighter">${time} • ${row[2]} • ${row[4]}</p>
                                </div>
                            </div>
                            <div class="text-right">
                                <p class="text-slate-900 font-extrabold text-lg italic">฿${row[3]}</p>
                            </div>
                        </div>
                    `;
                }).join('');
                
                totalDisplay.innerText = `฿${total.toLocaleString()}`;
            } catch (error) {
                console.error('Error:', error);
                list.innerHTML = '<p class="text-center text-red-400 text-sm">ไม่สามารถโหลดข้อมูลได้</p>';
            }
        }

        // 2. ฟังก์ชันส่งข้อมูล (POST)
        form.addEventListener('submit', async (e) => {
            e.preventDefault();
            
            // เตรียมปุ่ม
            submitBtn.disabled = true;
            submitBtn.innerHTML = '<i class="fas fa-circle-notch animate-spin mr-2"></i> กำลังบันทึก...';

            const payload = {
                customerName: document.getElementById('custName').value,
                service: document.getElementById('service').options[document.getElementById('service').selectedIndex].text.split(' (')[0],
                price: document.getElementById('service').value,
                staffName: document.getElementById('staffName').value
            };

            try {
                // ส่งข้อมูล
                await fetch(scriptURL, { 
                    method: 'POST', 
                    mode: 'no-cors', // เพิ่มเพื่อป้องกันปัญหา CORS
                    body: JSON.stringify(payload) 
                });

                showToast('✅ บันทึกข้อมูลสำเร็จ!');
                form.reset();
                
                // รอสักครู่แล้วโหลดข้อมูลใหม่ (เพราะ no-cors จะไม่รอ response)
                setTimeout(fetchHistory, 1500);

            } catch (error) {
                showToast('❌ เกิดข้อผิดพลาด กรุณาลองใหม่');
                console.error(error);
            } finally {
                submitBtn.disabled = false;
                submitBtn.innerHTML = '<span>บันทึกรายการ</span>';
            }
        });

        function showToast(msg) {
            const toast = document.getElementById('statusToast');
            toast.innerText = msg;
            toast.classList.remove('hidden');
            setTimeout(() => toast.classList.add('hidden'), 3000);
        }

        // โหลดข้อมูลทันทีที่เปิดแอป
        fetchHistory();
    </script>
</body>
</html>
