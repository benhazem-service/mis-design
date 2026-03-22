<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام إدارة خدمات الطباعة والكراء (3D Cloud)</title>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700;900&display=swap" rel="stylesheet">
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.8.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.8.1/firebase-firestore-compat.js"></script>

    <style>
        /* ===== متغيرات الألوان ===== */
        :root {
            --primary: #3b82f6; --primary-hover: #2563eb;
            --accent: #10b981; --accent-hover: #059669;
            --warning: #f97316; --danger: #ef4444; --success: #22c55e;
            --indigo: #6366f1; --whatsapp: #25D366;
            --text-main: #1e293b; --text-muted: #64748b;
            
            --glass-bg: rgba(255, 255, 255, 0.6);
            --glass-border: rgba(255, 255, 255, 0.8);
            --glass-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.07);
        }

        /* ===== تصفير الهوامش وتفعيل التجاوب ===== */
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Tajawal', sans-serif; }
        html, body { width: 100%; min-height: 100vh; color: var(--text-main); line-height: 1.6; overflow-x: hidden; }

        /* ===== الخلفية الديناميكية ===== */
        .bg-shapes { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; z-index: -1; overflow: hidden; background: #f0f4f8; }
        .shape { position: absolute; filter: blur(90px); border-radius: 50%; animation: float 20s infinite alternate ease-in-out; }
        .shape-1 { width: 500px; height: 500px; background: rgba(59, 130, 246, 0.3); top: -10%; right: -10%; }
        .shape-2 { width: 400px; height: 400px; background: rgba(16, 185, 129, 0.3); bottom: -10%; left: -10%; animation-delay: -5s; }
        .shape-3 { width: 600px; height: 600px; background: rgba(99, 102, 241, 0.2); top: 30%; left: 30%; animation-delay: -10s; }
        @keyframes float { 0% { transform: translate(0, 0) scale(1); } 100% { transform: translate(80px, 50px) scale(1.1); } }

        .glass-panel { background: var(--glass-bg); backdrop-filter: blur(16px); -webkit-backdrop-filter: blur(16px); border: 1px solid var(--glass-border); border-top: 1px solid rgba(255,255,255,1); border-left: 1px solid rgba(255,255,255,1); box-shadow: var(--glass-shadow); border-radius: 24px; }

        /* ===== إعدادات الطباعة (A5) ===== */
        .print-only { display: none; }
        @media print {
            @page { size: A5 portrait; margin: 0; }
            .no-print { display: none !important; }
            .print-only { display: block !important; width: 100vw; height: 100vh; background: white; position: absolute; top: 0; left: 0; color: black; overflow: hidden; }
            body, html { background-color: white; margin: 0; padding: 0; height: 100%; overflow: hidden; }
            .bg-shapes { display: none; }
            .receipt-wrapper { width: 148mm; height: 209mm; margin: 0 auto; padding: 8mm 10mm; box-sizing: border-box; display: flex; flex-direction: column; justify-content: space-between; }
            .receipt-header { text-align: center; border-bottom: 2px solid #000; padding-bottom: 5px; margin-bottom: 5px; }
            .print-logo { max-height: 55px; max-width: 180px; object-fit: contain; margin: 0 auto 5px; display: block; }
            .receipt-header h2 { margin-bottom: 2px; font-size: 20px; font-weight: 900; }
            .receipt-header p { font-size: 13px; margin: 0; font-weight: bold; color: #444; }
            .receipt-title { text-align: center; font-size: 16px; font-weight: 900; margin: 5px 0; padding: 5px; background-color: #f1f5f9 !important; border-radius: 6px; border: 1px solid #cbd5e1; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
            .receipt-table { width: 100%; font-size: 13px; margin-bottom: 5px; border-collapse: collapse; }
            .receipt-table th, .receipt-table td { padding: 4px 2px; border-bottom: 1px dotted #ccc; text-align: right; }
            .items-box { border: 2px solid #000; border-radius: 6px; padding: 8px; margin-bottom: 5px; flex-grow: 1; overflow: hidden; }
            .items-box h4 { margin-bottom: 5px; font-size: 14px; border-bottom: 1px dashed #000; padding-bottom: 3px; }
            .items-table { width: 100%; border-collapse: collapse; font-size: 13px; }
            .items-table th { padding: 4px 0; border-bottom: 1px solid #000; color:#000; font-weight:900; text-align: right;}
            .items-table td { padding: 4px 0; border-bottom: 1px dashed #eee; }
            .receipt-financials { border: 2px solid #000; background-color: #f8fafc !important; border-radius: 6px; padding: 8px; margin-bottom: 5px; font-size: 14px; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
            .receipt-financials div { display: flex; justify-content: space-between; margin-bottom: 3px; }
            .receipt-financials .rem { font-size: 16px; font-weight: 900; border-top: 1px dashed #000; padding-top: 5px; margin-top: 3px; }
            .receipt-footer { display: flex; justify-content: space-between; align-items: flex-end; padding-top: 5px; border-top: 2px solid #000; }
            .footer-info p { font-size: 12px; margin-bottom: 3px; }
            .qr-container { text-align: center; }
            .qr-container img { width: 60px; height: 60px; margin: 0 auto; border-radius: 6px;}
            .qr-container p { font-size: 10px; margin-top: 3px; font-weight: bold; }
        }

        /* ===== صفحة تتبع الزبون ===== */
        #customerTrackingView { display: none; min-height: 100vh; padding: 20px; position: relative; z-index: 10;}
        .track-container { max-width: 500px; margin: 5vh auto; background: var(--glass-bg); backdrop-filter: blur(20px); border-radius: 30px; box-shadow: var(--glass-shadow); border: 1px solid var(--glass-border); overflow: hidden; }
        .track-header { background: rgba(59, 130, 246, 0.85); color: white; padding: 30px 20px; text-align: center; }
        .track-header h1 { font-size: 24px; font-weight: 900; margin-bottom: 5px; text-shadow: 0 2px 4px rgba(0,0,0,0.2);}
        .track-logo { max-height: 70px; border-radius: 12px; margin-bottom: 10px; background: white; padding: 8px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);}
        .track-body { padding: 25px; }
        .track-status-box { text-align: center; padding: 20px; border-radius: 20px; margin-bottom: 20px; background: rgba(255,255,255,0.8); border: 2px solid transparent; box-shadow: 0 4px 15px rgba(0,0,0,0.05); transition: 0.3s; }
        .track-status-box.s-pending { border-color: rgba(100, 116, 139, 0.3); color: var(--text-main); }
        .track-status-box.s-processing { border-color: var(--primary); background: #eff6ff; color: var(--primary); box-shadow: 0 4px 20px rgba(59, 130, 246, 0.15);}
        .track-status-box.s-ready { border-color: var(--success); background: #f0fdf4; color: var(--success); box-shadow: 0 4px 20px rgba(34, 197, 94, 0.15);}
        .track-status-box.s-delivered { border-color: var(--indigo); background: #e0e7ff; color: var(--indigo); }
        .track-status-box h2 { font-size: 26px; font-weight: 900; margin-bottom: 5px; }
        .track-details { background: rgba(255,255,255,0.7); border: 1px solid var(--glass-border); border-radius: 16px; padding: 20px; margin-bottom: 15px; box-shadow: inset 0 2px 4px rgba(255,255,255,0.8); }
        .track-details h3 { font-size: 15px; color: var(--text-muted); margin-bottom: 15px; border-bottom: 1px dashed rgba(0,0,0,0.1); padding-bottom: 8px; }
        .track-row { display: flex; justify-content: space-between; font-size: 15px; margin-bottom: 10px; font-weight: 600; }
        .track-total-box { background: linear-gradient(135deg, #1e293b, #0f172a); color: white; padding: 20px; border-radius: 16px; display: flex; justify-content: space-between; align-items: center; box-shadow: 0 10px 25px rgba(0,0,0,0.2);}

        /* ===== لوحة الإدارة (100% عرض) ===== */
        .admin-view { display: block; position: relative; z-index: 10; width: 100%; }
        header { padding: 15px 2%; position: sticky; top: 0; z-index: 50; width: 100%; }
        .navbar { width: 100%; display: flex; justify-content: space-between; align-items: center; height: auto; padding: 15px 25px; }
        .logo { font-size: 24px; font-weight: 900; color: var(--primary); display: flex; align-items: center; gap: 10px; text-shadow: 0 2px 4px rgba(0,0,0,0.05);}
        .logo img { max-height: 45px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1);}
        .settings-btn { background: rgba(255,255,255,0.5); backdrop-filter: blur(10px); border: 1px solid rgba(255,255,255,0.8); padding: 10px 20px; border-radius: 12px; cursor: pointer; font-weight: bold; color: var(--text-main); display: flex; align-items: center; gap: 8px; transition: 0.3s; font-size: 15px; box-shadow: 0 4px 6px rgba(0,0,0,0.02);}
        .settings-btn:hover { background: white; transform: translateY(-2px); box-shadow: 0 6px 15px rgba(0,0,0,0.05); color: var(--primary); }
        
        main { padding: 10px 2%; width: 100%; display: flex; flex-direction: column; gap: 20px; }

        .search-bar { display:flex; align-items:center; padding: 12px 20px; border-radius: 20px; width: 100%; max-width: 800px; margin: 0 auto 10px;}
        .search-bar input { border:none; background:transparent; outline:none; width:100%; padding: 0 15px; font-size: 16px; font-weight: bold; color: var(--text-main); }

        .dashboard-cards { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 20px; width: 100%; }
        .dash-card { background: rgba(255, 255, 255, 0.5); backdrop-filter: blur(12px); border-radius: 24px; padding: 25px 20px; display: flex; align-items: center; gap: 15px; border: 1px solid rgba(255, 255, 255, 0.6); border-top-color: rgba(255,255,255,0.9); border-right-color: rgba(255,255,255,0.9); box-shadow: 8px 8px 20px rgba(0,0,0,0.03), -4px -4px 15px rgba(255,255,255,0.8); cursor: pointer; transition: 0.4s; width: 100%;}
        .dash-card:hover { transform: translateY(-8px) scale(1.02); box-shadow: 12px 15px 25px rgba(0,0,0,0.06), -6px -6px 20px rgba(255,255,255,1); }
        .card-icon { min-width: 65px; height: 65px; border-radius: 16px; display: flex; justify-content: center; align-items: center; background: white; color: var(--primary); box-shadow: inset 0 2px 4px rgba(0,0,0,0.05), 0 4px 10px rgba(0,0,0,0.05); font-size: 28px;}
        .card-info h3 { font-size: 16px; color: var(--text-muted); font-weight: 700; margin-bottom: 2px; }
        .card-info .number { font-size: 28px; font-weight: 900; color: var(--text-main); }
        .dash-card.new-order { background: linear-gradient(135deg, rgba(16, 185, 129, 0.8), rgba(5, 150, 105, 0.9)); border: none; box-shadow: 0 10px 25px rgba(16, 185, 129, 0.3); }
        .dash-card.new-order .card-icon { background: rgba(255, 255, 255, 0.25); color: white; box-shadow: none; }
        .dash-card.new-order h3 { color: rgba(255, 255, 255, 0.9); margin-bottom: 0; }
        .dash-card.new-order .number { color: white; font-size: 18px; margin-top: 5px; }
        .icon-warning { color: #d97706; } .icon-success { color: var(--accent); } .icon-indigo { color: var(--indigo); }

        /* ===== التقويم المزدوج ===== */
        .master-calendar-container { padding: 5px; width: 100%; }
        .master-header { display: flex; justify-content: space-between; align-items: center; padding: 25px 30px; border-bottom: 1px solid rgba(0,0,0,0.05); }
        .master-header h2 { font-size: 26px; color: var(--text-main); font-weight: 900; text-shadow: 0 2px 5px rgba(255,255,255,0.8);}
        .master-header button { background: rgba(255,255,255,0.6); box-shadow: 2px 2px 8px rgba(0,0,0,0.05), -2px -2px 8px rgba(255,255,255,0.8); border: 1px solid rgba(255,255,255,0.5); padding: 12px 30px; border-radius: 12px; cursor: pointer; font-weight: 800; font-size: 16px; transition: 0.3s; color: var(--text-main); }
        .master-header button:hover { background: white; transform: translateY(-2px); box-shadow: 4px 4px 12px rgba(0,0,0,0.08), -2px -2px 8px rgba(255,255,255,1); color: var(--primary); }
        
        .dual-calendars { display: grid; grid-template-columns: 1fr 2px 1fr; gap: 20px; padding: 30px; }
        .calendar-divider { background: linear-gradient(to bottom, transparent, rgba(59, 130, 246, 0.4), transparent); width: 2px; height: 90%; margin: auto; border-radius: 2px; }
        .calendar-box { background: transparent; padding: 0; width: 100%; }
        .month-title { text-align: center; font-size: 22px; color: var(--primary); margin-bottom: 25px; font-weight: 900; letter-spacing: 1px;}
        .weekdays { display: grid; grid-template-columns: repeat(7, 1fr); text-align: center; font-weight: 800; font-size: 16px; color: var(--text-muted); margin-bottom: 15px; }
        .days-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 15px; width: 100%; }
        
        .day { background: rgba(255, 255, 255, 0.4); border-radius: 16px; min-height: 110px; padding: 15px; text-align: right; font-weight: 700; font-size: 18px; box-shadow: 4px 4px 10px rgba(0,0,0,0.03), -4px -4px 10px rgba(255,255,255,0.6); border: 1px solid rgba(255,255,255,0.4); cursor: pointer; transition: 0.3s; position: relative; }
        .day:hover { background: rgba(255, 255, 255, 0.9); transform: translateY(-5px) scale(1.03); box-shadow: 8px 8px 20px rgba(0,0,0,0.06), -6px -6px 15px rgba(255,255,255,1); z-index: 5;}
        .day.empty { background: transparent; box-shadow: none; border: none; cursor: default; }
        .day.today::before { content: attr(data-day); position: absolute; top: 10px; right: 10px; background: linear-gradient(135deg, var(--primary), #8b5cf6); color: white; width: 35px; height: 35px; display: flex; align-items: center; justify-content: center; border-radius: 50%; font-weight: 900; font-size: 16px; z-index: 2; box-shadow: 0 4px 12px rgba(59, 130, 246, 0.4); }
        .day.today .day-number { visibility: hidden; }
        .day.has-orders { background: linear-gradient(135deg, rgba(16, 185, 129, 0.15), rgba(16, 185, 129, 0.05)); border: 1px solid rgba(16, 185, 129, 0.3); box-shadow: 4px 4px 12px rgba(16, 185, 129, 0.1), -4px -4px 10px rgba(255,255,255,0.7); }
        .day.has-orders:hover { background: linear-gradient(135deg, rgba(16, 185, 129, 0.25), rgba(16, 185, 129, 0.1)); }
        .order-badge { position: absolute; bottom: 10px; left: 10px; background: linear-gradient(135deg, #ef4444, #b91c1c); color: white; border-radius: 50%; width: 28px; height: 28px; display: flex; align-items: center; justify-content: center; font-size: 14px; font-weight: 900; box-shadow: 0 4px 10px rgba(239, 68, 68, 0.5); z-index: 5; }

        /* ===== النوافذ المنبثقة ===== */
        .modal-overlay { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(15, 23, 42, 0.4); backdrop-filter: blur(8px); z-index: 1000; display: flex; justify-content: center; align-items: center; opacity: 0; visibility: hidden; transition: 0.4s; }
        .modal-overlay.active { opacity: 1; visibility: visible; }
        .modal-box { background: rgba(255, 255, 255, 0.85); backdrop-filter: blur(25px); border: 1px solid rgba(255,255,255,0.9); box-shadow: 0 25px 50px -12px rgba(0,0,0,0.25); width: 95%; max-width: 900px; border-radius: 28px; transform: translateY(-30px) scale(0.95); transition: 0.4s; max-height: 90vh; overflow-y: auto; display: flex; flex-direction: column; }
        .modal-overlay.active .modal-box { transform: translateY(0) scale(1); }
        .fullscreen-modal { max-width: 98vw !important; width: 98%; height: 95vh; }
        
        .modal-header { padding: 25px 30px; border-bottom: 1px solid rgba(0,0,0,0.05); display: flex; justify-content: space-between; align-items: center; position: sticky; top: 0; background: rgba(255,255,255,0.9); backdrop-filter: blur(10px); z-index: 10; }
        .modal-header h2 { font-size: 22px; color: var(--text-main); font-weight: 900; display: flex; align-items: center; gap: 10px; }
        .close-btn { background: rgba(239, 68, 68, 0.1); width: 36px; height: 36px; border-radius: 50%; display: flex; justify-content: center; align-items: center; border: none; font-size: 20px; color: var(--danger); cursor: pointer; transition: 0.3s; }
        .close-btn:hover { background: var(--danger); color: white; transform: rotate(90deg);}
        .modal-body { padding: 30px; display: flex; flex-direction: column; gap: 25px; overflow-y: auto; }

        /* شاشة التتبع الملونة */
        .tracking-card { display: flex; justify-content: space-between; align-items: center; padding: 20px 25px; border-radius: 20px; margin-bottom: 20px; box-shadow: 4px 4px 15px rgba(0,0,0,0.03), -4px -4px 15px rgba(255,255,255,1); border: 1px solid rgba(255,255,255,0.6); border-right-width: 8px; background: rgba(255,255,255,0.6); backdrop-filter: blur(10px); transition: 0.3s;}
        .tracking-card:hover { transform: translateY(-3px); box-shadow: 6px 6px 20px rgba(0,0,0,0.05), -6px -6px 20px rgba(255,255,255,1); }
        .tracking-info h4 { margin-bottom: 8px; color: var(--text-main); font-size: 18px; font-weight: 900;}
        .tracking-info p { color: var(--text-muted); font-size: 14px; margin-bottom: 3px; font-weight: 600;}
        .tracking-status { display: flex; flex-direction: column; align-items: flex-end; gap: 12px; }
        .days-badge { padding: 8px 20px; border-radius: 20px; font-weight: 900; font-size: 14px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
        .track-red { border-right-color: var(--danger); } .track-red .days-badge { background: linear-gradient(135deg, var(--danger), #b91c1c); color: white; }
        .track-orange { border-right-color: var(--warning); } .track-orange .days-badge { background: linear-gradient(135deg, var(--warning), #c2410c); color: white; }
        .track-green { border-right-color: var(--success); } .track-green .days-badge { background: linear-gradient(135deg, var(--success), #15803d); color: white; }

        /* النماذج والإدخالات */
        .form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        .form-group { display: flex; flex-direction: column; gap: 8px; }
        .form-group.full-width { grid-column: span 2; }
        .form-group label { font-size: 15px; font-weight: 800; color: var(--text-main); text-shadow: 0 1px 2px rgba(255,255,255,0.8);}
        .form-group input, .form-group textarea, .form-group select { padding: 14px 18px; border-radius: 12px; font-size: 15px; outline: none; transition: 0.3s; background: rgba(255, 255, 255, 0.7); border: 1px solid rgba(255,255,255,0.5); box-shadow: inset 2px 2px 5px rgba(0,0,0,0.03), inset -2px -2px 5px rgba(255,255,255,1); font-weight: 600; color: var(--text-main); width: 100%; }
        .form-group input:focus, .form-group textarea:focus, .form-group select:focus { background: white; border-color: var(--primary); box-shadow: 0 0 0 4px rgba(59, 130, 246, 0.15); }
        .calc-row { background: rgba(255,255,255,0.5); padding: 20px; border-radius: 16px; border: 1px solid rgba(255,255,255,0.8); box-shadow: 4px 4px 15px rgba(0,0,0,0.02);}
        .calc-input { font-weight: 900; font-size: 18px !important; color: var(--primary); }
        .calc-readonly { background: rgba(226, 232, 240, 0.5) !important; color: var(--danger) !important; }
        .modal-footer { padding: 20px 30px; border-top: 1px solid rgba(0,0,0,0.05); display: flex; justify-content: flex-end; gap: 15px; position: sticky; bottom: 0; background: rgba(255,255,255,0.9); backdrop-filter: blur(10px); z-index: 10; border-radius: 0 0 28px 28px;}
        .btn-cancel { padding: 12px 25px; border: 1px solid var(--border); background: white; border-radius: 12px; font-weight: 800; cursor: pointer; transition: 0.2s; box-shadow: 0 2px 5px rgba(0,0,0,0.02);}
        .btn-cancel:hover { background: #f1f5f9; }
        .btn-save { padding: 12px 30px; background: linear-gradient(135deg, var(--accent), var(--accent-hover)); color: white; border: none; border-radius: 12px; font-weight: 900; cursor: pointer; transition: 0.3s; font-size: 16px; display: flex; align-items: center; gap: 8px; box-shadow: 0 4px 15px rgba(16, 185, 129, 0.3);}
        .btn-save:hover { transform: translateY(-2px); box-shadow: 0 6px 20px rgba(16, 185, 129, 0.4); }

        .btn-add-item { background: linear-gradient(135deg, var(--primary), var(--primary-hover)); color: white; border: none; padding: 12px 20px; border-radius: 12px; font-weight: bold; cursor: pointer; transition: 0.3s; height: 50px; box-shadow: 0 4px 15px rgba(59, 130, 246, 0.3); }
        .btn-add-item:hover { transform: translateY(-2px); box-shadow: 0 6px 20px rgba(59, 130, 246, 0.4); }
        .btn-add-rental { background: linear-gradient(135deg, var(--indigo), #4338ca); box-shadow: 0 4px 15px rgba(99, 102, 241, 0.3);}
        .cart-item { display: flex; justify-content: space-between; align-items: center; padding: 12px 18px; background: rgba(255,255,255,0.8); border: 1px solid var(--glass-border); border-radius: 12px; font-size: 15px; margin-bottom: 8px; font-weight: 600; box-shadow: 2px 2px 8px rgba(0,0,0,0.02);}
        .cart-item button { background: rgba(239, 68, 68, 0.1); width: 28px; height: 28px; border-radius: 50%; border: none; color: var(--danger); cursor: pointer; font-weight: bold; display: flex; justify-content: center; align-items: center; transition: 0.2s;}
        .cart-item button:hover { background: var(--danger); color: white; }

        .btn-print { background: linear-gradient(135deg, var(--text-main), #0f172a); color: white; padding: 12px 25px; border: none; border-radius: 12px; font-weight: bold; cursor: pointer; display: flex; align-items: center; gap: 8px; transition: 0.3s; box-shadow: 0 4px 15px rgba(0,0,0,0.2);}
        .btn-print:hover { transform: translateY(-2px); box-shadow: 0 6px 20px rgba(0,0,0,0.3); }
        .btn-print-small { background: white; border: 1px solid var(--border); padding: 8px 15px; border-radius: 8px; font-size: 13px; font-weight: 800; cursor: pointer; display: flex; align-items: center; gap: 5px; transition: 0.2s; box-shadow: 0 2px 4px rgba(0,0,0,0.02);}
        .btn-print-small:hover { background: #f8fafc; border-color: var(--primary); color: var(--primary); transform: translateY(-1px);}

        /* كروت الطلبات */
        .day-order-card { background: rgba(255,255,255,0.7); backdrop-filter: blur(10px); border: 1px solid rgba(255,255,255,0.8); padding: 20px; border-radius: 20px; margin-bottom: 20px; display: flex; flex-direction: column; gap: 15px; box-shadow: 4px 4px 15px rgba(0,0,0,0.02), -4px -4px 15px rgba(255,255,255,1); transition: 0.3s; }
        .day-order-card:hover { transform: translateY(-2px); box-shadow: 6px 6px 20px rgba(0,0,0,0.04), -6px -6px 20px rgba(255,255,255,1); }
        .order-top-row { display: flex; justify-content: space-between; align-items: flex-start; }
        .day-order-info h4 { color: var(--text-main); font-size: 18px; margin-bottom: 8px; font-weight: 900;}
        .day-order-info p { color: var(--text-muted); font-size: 15px; margin-bottom: 4px; font-weight: 600;}
        .day-order-price { text-align: left; background: rgba(255,255,255,0.9); padding: 12px 15px; border-radius: 12px; border: 1px solid rgba(0,0,0,0.05); box-shadow: inset 0 2px 4px rgba(0,0,0,0.02); min-width: 140px;}
        .day-order-price .total { font-weight: 900; color: var(--primary); font-size: 16px;}
        .day-order-price .rem { font-size: 14px; color: var(--danger); font-weight: 800;}
        .empty-day-msg { text-align: center; color: var(--text-muted); padding: 40px 10px; font-size: 18px; font-weight: 800; background: rgba(255,255,255,0.5); border-radius: 20px; border: 2px dashed rgba(0,0,0,0.1);}
        .order-id-badge { background: linear-gradient(135deg, var(--primary), #8b5cf6); color: white; padding: 4px 12px; border-radius: 8px; font-size: 14px; font-weight: 800; box-shadow: 0 2px 5px rgba(59, 130, 246, 0.3);}
        .status-bar { display: flex; align-items: center; justify-content: space-between; border-top: 1px dashed rgba(0,0,0,0.1); padding-top: 15px; }
        .status-select { padding: 8px 15px; border-radius: 10px; border: 1px solid var(--border); font-weight: 800; background: white; cursor: pointer; outline: none; box-shadow: 0 2px 5px rgba(0,0,0,0.02);}
        .btn-whatsapp { background: linear-gradient(135deg, var(--whatsapp), #16a34a); color: white; border: none; padding: 10px 20px; border-radius: 10px; font-weight: 900; cursor: pointer; display: flex; align-items: center; gap: 8px; transition: 0.3s; box-shadow: 0 4px 10px rgba(37,211,102,0.3);}
        
        /* تبريز الخانات في الإعدادات */
        .settings-section { margin-bottom: 30px; background: rgba(255,255,255,0.6); backdrop-filter: blur(10px); padding: 25px; border-radius: 20px; border: 1px solid rgba(255,255,255,0.8); box-shadow: 4px 4px 15px rgba(0,0,0,0.02); }
        .settings-section h3 { margin-bottom: 20px; color: var(--primary); font-size: 18px; font-weight: 900; display: flex; align-items: center; gap: 8px;}
        .settings-section input, .settings-section textarea, .settings-section select { background: #ffffff !important; border: 2px solid #cbd5e1 !important; color: #0f172a !important; box-shadow: 0 2px 4px rgba(0,0,0,0.05) !important; font-weight: bold; }
        .settings-section input:focus, .settings-section textarea:focus, .settings-section select:focus { border-color: var(--primary) !important; box-shadow: 0 0 0 4px rgba(59, 130, 246, 0.2) !important; }
        .service-list-item { display: flex; justify-content: space-between; align-items: center; padding: 12px 15px; background: white; border-radius: 12px; margin-bottom: 10px; border: 2px solid var(--border); box-shadow: 2px 2px 8px rgba(0,0,0,0.02); font-weight: 700;}
        .service-list-item button { background: rgba(239, 68, 68, 0.1); width: 30px; height: 30px; border-radius: 8px; border: none; color: var(--danger); cursor: pointer; font-weight: bold; font-size: 16px; display: flex; justify-content: center; align-items: center; transition: 0.2s;}
        .service-list-item button:hover { background: var(--danger); color: white; }
        .cloud-sync-status { font-size: 13px; display: flex; align-items: center; gap: 5px; color: var(--accent); font-weight: 900; background: rgba(209, 250, 229, 0.8); backdrop-filter: blur(5px); padding: 6px 15px; border-radius: 20px; margin-right: 10px; border: 1px solid rgba(209, 250, 229, 1); box-shadow: 0 2px 10px rgba(16, 185, 129, 0.1);}

        @media (max-width: 900px) {
            .form-row { grid-template-columns: 1fr; }
            .form-group.full-width { grid-column: span 1; }
            .dual-calendars { grid-template-columns: 1fr; gap: 30px; padding: 15px;}
            .calendar-divider { display: none; }
            .order-top-row { flex-direction: column; gap: 15px; }
            .day-order-price { text-align: right; width: 100%; display: flex; justify-content: space-between;}
            .status-bar { flex-direction: column; align-items: stretch; gap: 15px; }
            .tracking-card { flex-direction: column; align-items: stretch; gap: 15px; border-right-width: 1px; border-top-width: 6px;}
            .tracking-status { align-items: center; flex-direction: row; justify-content: space-between; border-top: 1px dashed rgba(0,0,0,0.1); padding-top: 15px; }
            .cloud-sync-status span { display: none; }
            .bg-shapes { display: none; }
            body { background: #f0f4f8; }
        }
    </style>
</head>
<body>

    <div class="bg-shapes">
        <div class="shape shape-1"></div>
        <div class="shape shape-2"></div>
        <div class="shape shape-3"></div>
    </div>

    <!-- 🌟 شاشة تتبع الزبون (عبر الباركود) 🌟 -->
    <div id="customerTrackingView" class="no-print">
        <div class="track-container">
            <div class="track-header">
                <img id="trackShopLogo" src="" alt="الشعار" class="track-logo" style="display:none; margin: 0 auto 15px;">
                <h1 id="trackShopName">المطبعة</h1>
                <p>تتبع حالة طلبك مباشرة في الوقت الفعلي</p>
            </div>
            <div class="track-body" id="trackBodyContent">
                <div style="text-align: center; padding: 50px 0;"><h3>جاري البحث عن الطلب... ⏳</h3></div>
            </div>
        </div>
    </div>

    <!-- 🖨️ قسم الطباعة A5 -->
    <div id="printReceiptArea" class="print-only">
        <div class="receipt-wrapper">
            <div>
                <div class="receipt-header">
                    <img id="r-shopLogo" src="" alt="الشعار" class="print-logo" style="display:none;">
                    <h2 id="r-shopName">اسم المطبعة</h2>
                    <p id="r-tagline">الشعار اللفظي</p>
                </div>
                <div class="receipt-title">وصل تسليم طلبية</div>
                <table class="receipt-table">
                    <tr><th>رقم الطلب:</th><td id="r-id" style="font-weight:900; font-size: 16px;">--</td></tr>
                    <tr><th>تاريخ التسجيل:</th><td id="r-today">--</td></tr>
                    <tr><th>العميل:</th><td id="r-name" style="font-weight:bold;">--</td></tr>
                    <tr><th>الهاتف:</th><td id="r-phone" dir="ltr" style="text-align: right;">--</td></tr>
                    <tr><th>موعد التسليم:</th><td id="r-date" style="font-weight:900; color:#000;">--</td></tr>
                    <tr id="r-notes-row" style="display:none;"><th>ملاحظة:</th><td id="r-notes">--</td></tr>
                </table>
                <div class="items-box">
                    <h4>الخدمات المطلوبة:</h4>
                    <div id="r-items-list"></div>
                </div>
            </div>
            
            <div>
                <div class="receipt-financials">
                    <div><span>المبلغ الإجمالي:</span> <strong id="r-total">--</strong></div>
                    <div><span>العربون المدفوع:</span> <strong id="r-deposit">--</strong></div>
                    <div class="rem"><span>الباقي للأداء:</span> <span id="r-remaining">--</span></div>
                </div>
                <div class="receipt-footer">
                    <div class="footer-info">
                        <p><strong>الهاتف:</strong> <span id="r-shopPhone"></span></p>
                        <p><strong>العنوان:</strong> <span id="r-shopAddress"></span></p>
                        <p style="margin-top: 5px; font-weight:bold; font-size: 14px;">شكراً لثقتكم بنا!</p>
                    </div>
                    <div style="display:flex; gap: 10px;">
                        <div class="qr-container" id="social-qr-container" style="display:none;">
                            <img id="r-socialQr" src="" alt="تابعنا">
                            <p>تابعنا</p>
                        </div>
                        <div class="qr-container">
                            <div id="r-qrcode"></div>
                            <p>امسح للتتبع</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- 💻 لوحة الإدارة -->
    <div class="admin-view no-print" id="adminAppView">
        
        <header class="glass-panel" style="margin: 20px 2%; border-radius: 20px; z-index: 100;">
            <div class="navbar" style="height: auto; padding: 15px;">
                <div class="logo">
                    <img id="mainAppLogo" src="" alt="Logo" style="display:none;">
                    <span id="mainAppNameText">مطبعتي</span>
                </div>
                <div style="display: flex; align-items: center;">
                    <div class="cloud-sync-status" id="cloudStatus">☁️ <span>متصل ومزامن</span></div>
                    <button class="settings-btn" onclick="openSettingsModal()">⚙️ الإعدادات</button>
                </div>
            </div>
        </header>

        <main>
            <!-- شريط البحث الذكي -->
            <div style="display: flex; justify-content: center; margin-bottom: 5px;">
                <div class="search-bar glass-panel" style="padding: 12px 20px; border-radius: 20px;">
                    <span style="font-size: 20px; opacity:0.7;">🔍</span>
                    <input type="text" id="searchInput" onkeypress="if(event.key === 'Enter') searchOrders()" placeholder="البحث برقم الهاتف، رقم الطلبية أو اسم الزبون...">
                    <button onclick="searchOrders()" style="background:var(--primary); color:white; border:none; padding:8px 20px; border-radius:12px; cursor:pointer; font-weight:bold; transition:0.3s;">بحث</button>
                </div>
            </div>

            <!-- الكروت العلوية -->
            <div class="dashboard-cards">
                <div class="dash-card new-order" onclick="openNewOrderModal()">
                    <div class="card-icon"><svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M12 5v14M5 12h14"/></svg></div>
                    <div class="card-info"><h3>إنشاء طلب / موعد</h3><div class="number">إضافة جديدة +</div></div>
                </div>
                <div class="dash-card" onclick="openTrackingModal()">
                    <div class="card-icon icon-warning"><svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline></svg></div>
                    <div class="card-info"><h3>قيد الانتظار / الإنجاز</h3><div class="number" id="statsPending">0 طلب</div></div>
                </div>
                <div class="dash-card" onclick="openListModal('delivered')">
                    <div class="card-icon icon-success"><svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"></path><polyline points="22 4 12 14.01 9 11.01"></polyline></svg></div>
                    <div class="card-info"><h3>طلبات مسلمة</h3><div class="number" id="statsDelivered">0 طلب</div></div>
                </div>
                <div class="dash-card" onclick="openListModal('rentals')">
                    <div class="card-icon icon-indigo"><svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><rect x="3" y="3" width="18" height="18" rx="2"/><path d="M3 9h18M9 21V9"/></svg></div>
                    <div class="card-info"><h3>لوازم في الكراء</h3><div class="number" id="statsRented">0 طلب</div></div>
                </div>
            </div>

            <!-- التقويم המزدوج -->
            <div class="master-calendar-container glass-panel">
                <div class="master-header">
                    <button id="nextMonth">❮ التالي</button>
                    <h2 id="masterYearDisplay">2026</h2>
                    <button id="prevMonth">السابق ❯</button>
                </div>
                <div class="dual-calendars">
                    <div class="calendar-box">
                        <h3 class="month-title" id="month1Title">--</h3>
                        <div class="weekdays"><div>الأحد</div><div>الإثنين</div><div>الثلاثاء</div><div>الأربعاء</div><div>الخميس</div><div>الجمعة</div><div>السبت</div></div>
                        <div class="days-grid" id="daysGrid1"></div>
                    </div>
                    <div class="calendar-divider"></div>
                    <div class="calendar-box">
                        <h3 class="month-title" id="month2Title">--</h3>
                        <div class="weekdays"><div>الأحد</div><div>الإثنين</div><div>الثلاثاء</div><div>الأربعاء</div><div>الخميس</div><div>الجمعة</div><div>السبت</div></div>
                        <div class="days-grid" id="daysGrid2"></div>
                    </div>
                </div>
            </div>
        </main>
    </div>

    <!-- ==============================================
         النوافذ المنبثقة (Modals)
         ============================================== -->

    <!-- إضافة / تعديل طلب -->
    <div class="modal-overlay no-print" id="orderModal">
        <div class="modal-box">
            <div class="modal-header">
                <h2 id="orderModalTitle">إضافة طلب <span class="order-id-badge" id="displayOrderId">#ORD-0000</span></h2>
                <button class="close-btn" onclick="closeModal('orderModal')">×</button>
            </div>
            <div class="modal-body">
                <div class="form-row">
                    <div class="form-group"><label>الاسم / الجهة *</label><input type="text" id="inpName"></div>
                    <div class="form-group"><label>رقم الهاتف *</label><input type="tel" id="inpPhone" placeholder="06XXXXXXXX"></div>
                </div>
                <div class="form-row">
                    <div class="form-group"><label>موعد تسليم الطلب للزبون *</label><input type="date" id="inpDate"></div>
                    <div class="form-group"><label>ملاحظات (تظهر بالوصل)</label><input type="text" id="inpNotes" placeholder="أي ملاحظة إضافية..."></div>
                </div>

                <div class="form-group full-width" style="background: rgba(255,255,255,0.5); padding: 20px; border-radius: 16px; border: 1px solid rgba(255,255,255,0.8); box-shadow: inset 0 2px 10px rgba(0,0,0,0.02);">
                    <label style="color: var(--primary); margin-bottom: 10px; display:block; font-size:16px;">🖨️ إضافة خدمات الطباعة</label>
                    <div style="display: flex; gap: 12px; align-items: center; flex-wrap: wrap;">
                        <select id="selCategory" style="flex: 1.5; min-width: 120px;" onchange="filterServicesByCategory()"></select>
                        <select id="selService" style="flex: 2; min-width: 150px;"><option value="">اختر الخدمة...</option></select>
                        <input type="number" id="qtyService" min="1" value="1" style="width: 90px;" placeholder="العدد">
                        <button class="btn-add-item" type="button" onclick="addServiceToOrder()">إضافة +</button>
                    </div>
                    <div id="cartServicesList" style="margin-top: 15px; display: flex; flex-direction: column; gap: 8px;"></div>
                </div>
                
                <div class="form-group full-width" style="background: rgba(99, 102, 241, 0.05); padding: 20px; border-radius: 16px; border: 1px solid rgba(99, 102, 241, 0.2);">
                    <label style="color: var(--indigo); margin-bottom: 10px; display:block; font-size:16px;">📦 إضافة لوازم للكراء</label>
                    <div style="display: flex; gap: 12px; align-items: center; flex-wrap: wrap;">
                        <select id="selRental" style="flex: 2; min-width: 150px;"></select>
                        <input type="number" id="qtyRental" min="1" value="1" style="width: 90px;">
                        <input type="date" id="dateRentalReturn" style="flex: 1; min-width: 140px;" title="تاريخ الاسترجاع">
                        <button class="btn-add-item btn-add-rental" type="button" onclick="addRentalToOrder()">إضافة +</button>
                    </div>
                    <div id="cartRentalsList" style="margin-top: 15px; display: flex; flex-direction: column; gap: 8px;"></div>
                </div>

                <div class="form-row calc-row">
                    <div class="form-group"><label>المبلغ الإجمالي</label><input type="number" id="inpTotal" class="calc-input" oninput="calculateRemaining()"></div>
                    <div class="form-group"><label>العربون</label><input type="number" id="inpDeposit" class="calc-input" oninput="calculateRemaining()"></div>
                    <div class="form-group full-width"><label>الباقي</label><input type="number" id="inpRemaining" class="calc-input calc-readonly" readonly></div>
                </div>
            </div>
            <div class="modal-footer">
                <button class="btn-cancel" onclick="closeModal('orderModal')">إلغاء</button>
                <button class="btn-save" onclick="saveNewOrder()" id="btnSaveOrderReal">حفظ الطلب ✓</button>
            </div>
        </div>
    </div>

    <!-- نافذة نجاح الحفظ -->
    <div class="modal-overlay no-print" id="successModal">
        <div class="modal-box" style="text-align: center; max-width: 450px; padding: 40px 30px; margin: auto;">
             <div style="font-size: 60px; margin-bottom: 15px; text-shadow: 0 10px 20px rgba(34, 197, 94, 0.3);">✅</div>
             <h2 style="color: var(--text-main); margin-bottom: 10px; font-weight:900;">تم الحفظ بنجاح!</h2>
             <p style="color: var(--text-muted); margin-bottom: 30px; font-size:16px;">رقم الطلب: <span id="successOrderId" class="order-id-badge" style="font-size:16px;"></span></p>
             <div style="display: flex; flex-direction: column; gap: 15px; justify-content: center;">
                 <button class="btn-print" style="width: 100%; justify-content: center; height: 50px;" onclick="printReceipt(currentSavedOrderId)">🖨️ طباعة وصل التسليم (A5)</button>
                 <button class="btn-cancel" style="width: 100%; height: 50px;" onclick="closeModal('successModal')">العودة للوحة التحكم</button>
             </div>
        </div>
    </div>

    <!-- قائمة الطلبات العامة (يومية / بحث / حالات) -->
    <div class="modal-overlay no-print" id="ordersListModal">
        <div class="modal-box">
            <div class="modal-header">
                <h2 id="ordersListTitle">📅 قائمة الطلبات</h2>
                <button class="close-btn" onclick="closeModal('ordersListModal')">×</button>
            </div>
            <div class="modal-body" id="ordersListContainer"></div>
            <div class="modal-footer" style="justify-content: center; display: none;" id="btnDaySpecificAdd">
                <button class="btn-save" style="width: 100%; height:55px; justify-content:center; font-size:18px;" onclick="openNewOrderWithDate()">+ إضافة طلب في هذا اليوم</button>
            </div>
        </div>
    </div>

    <!-- 📊 شاشة تتبع المواعيد -->
    <div class="modal-overlay no-print" id="trackingModal">
        <div class="modal-box fullscreen-modal">
            <div class="modal-header">
                <h2>📊 تتبع المواعيد والآجال بذكاء</h2>
                <button class="close-btn" onclick="closeModal('trackingModal')">×</button>
            </div>
            <div class="modal-body" style="background: rgba(255,255,255,0.4);">
                <div style="margin-bottom: 20px; display: flex; gap: 15px; flex-wrap: wrap;">
                    <span style="background:linear-gradient(135deg, var(--danger), #b91c1c); color:white; padding:8px 15px; border-radius:12px; font-size:14px; font-weight:bold; box-shadow:0 4px 10px rgba(239, 68, 68, 0.3);">أقل من 5 أيام 🟥</span>
                    <span style="background:linear-gradient(135deg, var(--warning), #c2410c); color:white; padding:8px 15px; border-radius:12px; font-size:14px; font-weight:bold; box-shadow:0 4px 10px rgba(249, 115, 22, 0.3);">أقل من 10 أيام 🟧</span>
                    <span style="background:linear-gradient(135deg, var(--success), #15803d); color:white; padding:8px 15px; border-radius:12px; font-size:14px; font-weight:bold; box-shadow:0 4px 10px rgba(34, 197, 94, 0.3);">أكثر من 10 أيام 🟩</span>
                </div>
                <div id="trackingListContainer"></div>
            </div>
        </div>
    </div>

    <!-- اختيار رسالة الواتساب -->
    <div class="modal-overlay no-print" id="waSelectModal">
        <div class="modal-box" style="max-width: 400px;">
            <div class="modal-header">
                <h2>💬 اختر الرسالة</h2>
                <button class="close-btn" onclick="closeModal('waSelectModal')">×</button>
            </div>
            <div class="modal-body" id="waSelectOptions" style="display:flex; flex-direction:column; gap:10px;"></div>
        </div>
    </div>

    <!-- ⚙️ الإعدادات والمزامنة -->
    <div class="modal-overlay no-print" id="settingsModal">
        <div class="modal-box">
            <div class="modal-header">
                <h2>⚙️ إعدادات النظام والمزامنة السحابية</h2>
                <button class="close-btn" onclick="closeModal('settingsModal')">×</button>
            </div>
            <div class="modal-body" style="background: rgba(255,255,255,0.3);">
                
                <div class="settings-section">
                    <h3>🏢 معلومات المطبعة والصور</h3>
                    <div class="form-row">
                        <!-- اللوغو الأساسي -->
                        <div class="form-group full-width" style="margin-bottom: 15px; border: 2px dashed rgba(59, 130, 246, 0.4); padding: 25px; border-radius: 16px; text-align: center; background: rgba(255,255,255,0.8); transition:0.3s;">
                            <label style="color: var(--primary); cursor: pointer; display: flex; flex-direction: column; align-items: center; gap: 10px;">
                                <span style="font-size: 36px;">🖼️</span>
                                <span style="font-size: 16px; font-weight:bold;">إدراج لوغو المطبعة (يظهر في الوصل)</span>
                                <input type="file" id="setShopLogoFile" accept="image/*" style="display: none;">
                            </label>
                            <img id="setShopLogoPreview" src="" style="display: none; max-height: 80px; margin: 15px auto 0; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1);">
                        </div>
                        <!-- صورة تابعنا -->
                        <div class="form-group full-width" style="margin-bottom: 15px; border: 2px dashed rgba(34, 197, 94, 0.4); padding: 25px; border-radius: 16px; text-align: center; background: rgba(255,255,255,0.8); transition:0.3s;">
                            <label style="color: var(--success); cursor: pointer; display: flex; flex-direction: column; align-items: center; gap: 10px;">
                                <span style="font-size: 36px;">📱</span>
                                <span style="font-size: 16px; font-weight:bold;">إدراج كيو آر (تابعنا) لمواقع التواصل</span>
                                <input type="file" id="setSocialQrFile" accept="image/*" style="display: none;">
                            </label>
                            <img id="setSocialQrPreview" src="" style="display: none; max-height: 80px; margin: 15px auto 0; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1);">
                        </div>
                    </div>
                    <div class="form-row">
                        <div class="form-group"><label>اسم المطبعة</label><input type="text" id="setShopName"></div>
                        <div class="form-group"><label>الشعار اللفظي</label><input type="text" id="setShopTagline"></div>
                        <div class="form-group"><label>رقم الهاتف</label><input type="text" id="setShopPhone" dir="ltr" style="text-align: right;"></div>
                        <div class="form-group"><label>العنوان</label><input type="text" id="setShopAddress"></div>
                    </div>
                </div>

                <div class="settings-section">
                    <h3>🏷️ خدمات الطباعة والتصميم</h3>
                    <div class="form-row" style="align-items: end; margin-bottom: 20px;">
                        <div class="form-group">
                            <label>صنف الخدمة</label>
                            <div style="display:flex; gap:8px;">
                                <select id="newServiceCategory" style="flex:1;"></select>
                                <button class="btn-print-small" onclick="promptNewCategory()" title="إضافة صنف جديد">+ صنف</button>
                            </div>
                        </div>
                        <div class="form-group"><label>اسم الخدمة</label><input type="text" id="newServiceName" placeholder="مثال: كروت شخصية"></div>
                        <div class="form-group"><label>الثمن (درهم)</label><input type="number" id="newServicePrice"></div>
                        <button class="btn-save" style="height: 50px; justify-content: center;" onclick="addNewService()">إضافة +</button>
                    </div>
                    <div id="settingsServicesList"></div>
                </div>

                <div class="settings-section">
                    <h3 style="color: var(--indigo);">📦 لوازم الكراء</h3>
                    <div class="form-row" style="align-items: end; margin-bottom: 20px;">
                        <div class="form-group"><label>اسم الأداة</label><input type="text" id="newRentalName"></div>
                        <div class="form-group"><label>الثمن (درهم)</label><input type="number" id="newRentalPrice"></div>
                        <button class="btn-save" style="background-color: var(--indigo); height: 50px; justify-content: center;" onclick="addNewRental()">إضافة +</button>
                    </div>
                    <div id="settingsRentalsList"></div>
                </div>
                
                <div class="settings-section">
                    <h3>💬 رسائل الواتس آب (النماذج الجاهزة)</h3>
                    <div class="form-row" style="align-items: end; margin-bottom: 20px;">
                        <div class="form-group"><label>عنوان الرسالة</label><input type="text" id="newWaTitle" placeholder="مثال: طلب جاهز"></div>
                        <div class="form-group"><label>نص الرسالة (استخدم [الاسم] و[رقم_الطلب])</label><textarea id="newWaMsg" rows="2" placeholder="مرحباً [الاسم]..."></textarea></div>
                        <button class="btn-save" style="height: 50px; justify-content: center;" onclick="addNewWaTemplate()">إضافة رسالة +</button>
                    </div>
                    <div id="settingsWaList"></div>
                </div>

                <div class="settings-section">
                    <h3>🛡️ الأمان وحماية البيانات</h3>
                    <div class="form-row">
                        <div class="form-group">
                            <label>كلمة مرور الحذف والتعديل (مهم جداً)</label>
                            <input type="password" id="setDeletePassword" placeholder="الافتراضي: 1234">
                        </div>
                    </div>
                </div>

                <div class="settings-section">
                    <h3>💾 قاعدة البيانات واسترداد الفايربيس</h3>
                    <div style="display: flex; gap: 15px;">
                        <button class="btn-export" style="height:45px;" onclick="exportDataToJSON()">⬇️ تصدير البيانات (JSON)</button>
                        <button class="btn-import" style="height:45px;" onclick="document.getElementById('importFile').click()">⬆️ استرداد ومزامنة</button>
                        <input type="file" id="importFile" accept=".json" style="display: none;" onchange="importDataFromJSON(event)">
                    </div>
                </div>
            </div>
            <div class="modal-footer">
                <button class="btn-cancel" onclick="closeModal('settingsModal')">إغلاق</button>
                <button class="btn-save" onclick="saveAllSettings()">حفظ الإعدادات ورفع للسحابة ☁️</button>
            </div>
        </div>
    </div>

    <!-- ==============================================
         أكواد الجافاسكريبت والفايربيس النظيفة والمدمجة
         ============================================== -->
    <script>
        // ----------------------------------------------------
        // 1. Firebase Initialization
        // ----------------------------------------------------
        const firebaseConfig = {
            apiKey: "AIzaSyDPBpDb2ECzxDlMH02x6zeIhL3vlDMelSk",
            authDomain: "mawa3id-mis-designe.firebaseapp.com",
            projectId: "mawa3id-mis-designe",
            storageBucket: "mawa3id-mis-designe.firebasestorage.app",
            messagingSenderId: "1083427888752",
            appId: "1:1083427888752:web:4198f35efe3475a1461eb5"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();
        db.enablePersistence().catch(err => console.warn("Offline DB Error", err));

        // ----------------------------------------------------
        // 2. Global Variables & State
        // ----------------------------------------------------
        let ordersData =[];
        let shopSettings = { 
            name: "مطبعتي", tagline: "لخدمات الطباعة", phone: "0600000000", address: "المدينة", 
            logo: "", socialQr: "", deletePassword: "1234",
            waTemplates:[ { title: "طلب جاهز", msg: "مرحباً [الاسم]، طلبكم رقم[رقم_الطلب] جاهز الآن للاستلام." } ]
        };
        let categoriesData =["مطبوعات ورقية", "تصميم", "هدايا إعلانية"]; 
        let servicesData =[ { category: "مطبوعات ورقية", name: "كروت شخصية", price: 100 } ]; 
        let rentalItemsData =[ { name: "حامل رول آب", price: 50 } ];
        
        let cartServices = []; let cartRentals =[]; 
        let currentSavedOrderId = ""; let currentViewContext = {}; 
        let tempLogoBase64 = ""; let tempSocialQrBase64 = "";
        let isEditing = false; let editingOrderId = null; let currentWaOrderId = "";
        const daysArabic =["الأحد", "الإثنين", "الثلاثاء", "الأربعاء", "الخميس", "الجمعة", "السبت"];

        // ----------------------------------------------------
        // 3. Helper Functions
        // ----------------------------------------------------
        function getIsoDate(year, month, day) { return `${year}-${String(month + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`; }
        function getTodayIsoString() { let d = new Date(); return getIsoDate(d.getFullYear(), d.getMonth(), d.getDate()); }
        function openModal(id) { document.getElementById(id).classList.add('active'); }
        function closeModal(id) { document.getElementById(id).classList.remove('active'); }
        function verifyDelete() { let pwd = prompt("أدخل كلمة المرور لتأكيد العملية:"); return pwd === (shopSettings.deletePassword || "1234"); }

        // ----------------------------------------------------
        // 4. Image Uploading
        // ----------------------------------------------------
        function handleImageUpload(inputId, previewId, callback) {
            document.getElementById(inputId).addEventListener('change', function(e) {
                const file = e.target.files[0];
                if (file) {
                    const reader = new FileReader();
                    reader.onload = function(evt) {
                        const img = new Image();
                        img.onload = function() {
                            const canvas = document.createElement('canvas'); const MAX_WIDTH = 300; let scaleSize = 1;
                            if(img.width > MAX_WIDTH) scaleSize = MAX_WIDTH / img.width;
                            canvas.width = img.width * scaleSize; canvas.height = img.height * scaleSize;
                            const ctx = canvas.getContext('2d'); ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                            let b64 = canvas.toDataURL('image/jpeg', 0.8);
                            document.getElementById(previewId).src = b64; document.getElementById(previewId).style.display = 'block';
                            callback(b64);
                        }
                        img.src = evt.target.result;
                    }; reader.readAsDataURL(file);
                }
            });
        }
        handleImageUpload('setShopLogoFile', 'setShopLogoPreview', (b64) => tempLogoBase64 = b64);
        handleImageUpload('setSocialQrFile', 'setSocialQrPreview', (b64) => tempSocialQrBase64 = b64);

        // ----------------------------------------------------
        // 5. Core App Logic (Admin vs Customer)
        // ----------------------------------------------------
        window.addEventListener('DOMContentLoaded', () => {
            const urlParams = new URLSearchParams(window.location.search);
            if(urlParams.get('track')) loadCustomerTrackingView(urlParams.get('track'));
            else loadAdminAppView();
        });

        function loadCustomerTrackingView(trackOrderId) {
            document.getElementById('adminAppView').style.display = 'none';
            document.getElementById('customerTrackingView').style.display = 'block';

            db.collection("config").doc("main").get().then((doc) => {
                if(doc.exists && doc.data().shopSettings) {
                    let st = doc.data().shopSettings;
                    document.getElementById('trackShopName').innerText = st.name;
                    if(st.logo) { document.getElementById('trackShopLogo').src = st.logo; document.getElementById('trackShopLogo').style.display = 'block'; }
                }
            });

            db.collection("orders").doc(trackOrderId).onSnapshot((doc) => {
                let bodyBox = document.getElementById('trackBodyContent');
                if(doc.exists) {
                    let o = doc.data();
                    let statusColorClass = "s-pending"; let statusEmoji = "⏳";
                    if(o.status === "قيد الإنجاز") { statusColorClass = "s-processing"; statusEmoji = "⚙️"; }
                    if(o.status === "جاهز") { statusColorClass = "s-ready"; statusEmoji = "✅"; }
                    if(o.status === "تم التسليم") { statusColorClass = "s-delivered"; statusEmoji = "📦"; }

                    let typesHTML = o.types.length > 0 ? o.types.map(t=>`<div>${t.name} <span style="color:#888;">(x${t.qty})</span> <b style="float:left;">${t.subtotal} د.م</b></div>`).join('') : '<span style="color:#888">لا توجد خدمات مطبوعة</span>';
                    let rentalsHTML = o.rentals.length > 0 ? o.rentals.map(r=>`<div>📦 ${r.name} <span style="color:#888;">(x${r.qty})</span> <b style="float:left;">${r.subtotal} د.م</b><br><small style="color:var(--danger)">إرجاع: ${r.returnDate}</small></div>`).join('<br>') : '';

                    bodyBox.innerHTML = `
                        <div class="track-status-box ${statusColorClass}"><h2>${statusEmoji} ${o.status}</h2><p style="margin-top:5px; font-weight:bold;">رقم الطلب: ${o.id}</p></div>
                        <div class="track-details"><h3>👤 معلومات الزبون</h3><div class="track-row"><span>الاسم:</span> <b>${o.name}</b></div><div class="track-row"><span>موعد التسليم:</span> <b dir="ltr" style="color:var(--primary)">${o.date}</b></div></div>
                        <div class="track-details"><h3>🖨️ تفاصيل الطلب</h3><div style="font-weight:bold; font-size:15px; border-bottom:1px dashed #ccc; padding-bottom:10px; margin-bottom:10px;">${typesHTML}</div>
                        <div class="track-row"><span>الإجمالي:</span> <b>${o.total} د.م</b></div><div class="track-row"><span>العربون:</span> <b>${o.deposit} د.م</b></div></div>
                        ${o.rentals.length > 0 ? `<div class="track-details" style="border-color:#c7d2fe; background:#eef2ff;"><h3 style="color:var(--indigo); border-color:#c7d2fe;">📦 لوازم الكراء</h3><div style="font-weight:bold; font-size:15px; color:var(--indigo);">${rentalsHTML}</div></div>` : ''}
                        <div class="track-total-box"><div style="font-size:14px;">المبلغ الباقي للدفع:</div><div style="font-size:20px; font-weight:900;">${o.remaining} درهم</div></div>
                    `;
                } else { bodyBox.innerHTML = `<div class="track-status-box s-pending" style="color:var(--danger); border-color:var(--danger);"><h2>❌ عذراً</h2><p>رقم الطلب غير صحيح أو تم حذفه.</p></div>`; }
            });
        }

        function loadAdminAppView() {
            db.collection("config").doc("main").onSnapshot((doc) => {
                if (doc.exists) {
                    let data = doc.data();
                    shopSettings = data.shopSettings || shopSettings; servicesData = data.servicesData || servicesData; rentalItemsData = data.rentalItemsData || rentalItemsData; categoriesData = data.categoriesData || categoriesData;
                    if(!shopSettings.waTemplates) shopSettings.waTemplates = [{ title: "طلب جاهز", msg: shopSettings.waTemplate || "مرحباً [الاسم]..." }];
                    tempLogoBase64 = shopSettings.logo || ""; tempSocialQrBase64 = shopSettings.socialQr || "";
                    updateAppUI();
                    
                    if(document.getElementById('settingsModal').classList.contains('active')) {
                        document.getElementById('setShopName').value = shopSettings.name; document.getElementById('setShopTagline').value = shopSettings.tagline; document.getElementById('setShopPhone').value = shopSettings.phone; document.getElementById('setShopAddress').value = shopSettings.address; document.getElementById('setDeletePassword').value = shopSettings.deletePassword || "1234";
                        if(tempLogoBase64){ document.getElementById('setShopLogoPreview').src = tempLogoBase64; document.getElementById('setShopLogoPreview').style.display = 'block'; }
                        if(tempSocialQrBase64){ document.getElementById('setSocialQrPreview').src = tempSocialQrBase64; document.getElementById('setSocialQrPreview').style.display = 'block'; }
                        populateSettingsCategories(); renderServicesListInSettings(); renderRentalsListInSettings(); renderWaTemplatesInSettings();
                    }
                } else { syncSettingsToFirebase(); }
            });

            db.collection("orders").onSnapshot((snapshot) => {
                ordersData =[]; snapshot.forEach((doc) => { ordersData.push(doc.data()); });
                updateDashboardStats(); renderDualCalendars(currentMonth, currentYear);
                if(document.getElementById('trackingModal').classList.contains('active')) renderTrackingList();
                if(document.getElementById('ordersListModal').classList.contains('active')) refreshCurrentListModal();
            });
            renderDualCalendars(currentMonth, currentYear);
        }

        function updateAppUI() {
            if(shopSettings.logo) { document.getElementById('mainAppLogo').src = shopSettings.logo; document.getElementById('mainAppLogo').style.display = 'block'; document.getElementById('mainAppNameText').style.display = 'none'; } 
            else { document.getElementById('mainAppLogo').style.display = 'none'; document.getElementById('mainAppNameText').style.display = 'inline'; document.getElementById('mainAppNameText').innerText = shopSettings.name; }
        }

        function syncSettingsToFirebase() { db.collection("config").doc("main").set({ shopSettings: shopSettings, servicesData: servicesData, rentalItemsData: rentalItemsData, categoriesData: categoriesData }); }

        // ----------------------------------------------------
        // 6. Settings Modal Logic
        // ----------------------------------------------------
        function openSettingsModal() { 
            document.getElementById('setShopName').value = shopSettings.name; document.getElementById('setShopTagline').value = shopSettings.tagline; document.getElementById('setShopPhone').value = shopSettings.phone; document.getElementById('setShopAddress').value = shopSettings.address; document.getElementById('setDeletePassword').value = shopSettings.deletePassword || "1234";
            tempLogoBase64 = shopSettings.logo || ""; tempSocialQrBase64 = shopSettings.socialQr || "";
            if(tempLogoBase64) { document.getElementById('setShopLogoPreview').src = tempLogoBase64; document.getElementById('setShopLogoPreview').style.display = 'block'; }
            if(tempSocialQrBase64) { document.getElementById('setSocialQrPreview').src = tempSocialQrBase64; document.getElementById('setSocialQrPreview').style.display = 'block'; }
            populateSettingsCategories(); renderServicesListInSettings(); renderRentalsListInSettings(); renderWaTemplatesInSettings();
            openModal('settingsModal'); 
        }

        function saveAllSettings() { 
            shopSettings.name = document.getElementById('setShopName').value; shopSettings.tagline = document.getElementById('setShopTagline').value; shopSettings.phone = document.getElementById('setShopPhone').value; shopSettings.address = document.getElementById('setShopAddress').value; 
            shopSettings.deletePassword = document.getElementById('setDeletePassword').value || "1234";
            shopSettings.logo = tempLogoBase64; shopSettings.socialQr = tempSocialQrBase64;
            syncSettingsToFirebase(); closeModal('settingsModal'); alert("تم الحفظ بنجاح!"); 
        }

        function populateSettingsCategories() { let sel = document.getElementById('newServiceCategory'); sel.innerHTML = ""; categoriesData.forEach(c => { sel.innerHTML += `<option value="${c}">${c}</option>`; }); }
        function promptNewCategory() { let c = prompt("أدخل اسم الصنف الجديد:"); if(c && c.trim() !== "") { if(!categoriesData.includes(c.trim())) { categoriesData.push(c.trim()); syncSettingsToFirebase(); } populateSettingsCategories(); document.getElementById('newServiceCategory').value = c.trim(); } }
        
        function renderServicesListInSettings() { const list = document.getElementById('settingsServicesList'); list.innerHTML = ""; servicesData.forEach((srv, index) => { let catBadge = srv.category ? `<span style="background:#e2e8f0;font-size:11px;padding:3px 6px;border-radius:4px;margin-left:5px;">${srv.category}</span>` : ''; list.innerHTML += `<div class="service-list-item"><div>${catBadge}<strong>${srv.name}</strong> (${srv.price} د.م)</div><button onclick="removeService(${index})">✖</button></div>`; }); }
        function addNewService() { let c = document.getElementById('newServiceCategory').value || 'عام'; let n = document.getElementById('newServiceName').value; let p = parseFloat(document.getElementById('newServicePrice').value) || 0; if(n && c) { servicesData.push({ category: c, name: n, price: p }); document.getElementById('newServiceName').value = ""; syncSettingsToFirebase(); } else { alert("يرجى اختيار الصنف والاسم."); } }
        function removeService(index) { if(!verifyDelete()) return; if(confirm("تأكيد الحذف؟")) { servicesData.splice(index, 1); syncSettingsToFirebase(); } }

        function renderRentalsListInSettings() { const list = document.getElementById('settingsRentalsList'); list.innerHTML = ""; rentalItemsData.forEach((r, index) => { list.innerHTML += `<div class="service-list-item"><div><strong>${r.name}</strong> <span style="color:var(--indigo)">(${r.price} د.م)</span></div><button onclick="removeRental(${index})">✖</button></div>`; }); }
        function addNewRental() { let n = document.getElementById('newRentalName').value; let p = parseFloat(document.getElementById('newRentalPrice').value) || 0; if(n) { rentalItemsData.push({ name: n, price: p }); document.getElementById('newRentalName').value = ""; syncSettingsToFirebase(); } }
        function removeRental(index) { if(!verifyDelete()) return; if(confirm("تأكيد الحذف؟")) { rentalItemsData.splice(index, 1); syncSettingsToFirebase(); } }

        function renderWaTemplatesInSettings() { const list = document.getElementById('settingsWaList'); list.innerHTML = ""; (shopSettings.waTemplates ||[]).forEach((tpl, index) => { list.innerHTML += `<div class="service-list-item" style="flex-direction:column; align-items:start; gap:5px;"><div style="display:flex; justify-content:space-between; width:100%;"><strong>${tpl.title}</strong><button onclick="removeWaTemplate(${index})">✖</button></div><div style="font-size:13px; color:var(--text-muted);">${tpl.msg}</div></div>`; }); }
        function addNewWaTemplate() { let t = document.getElementById('newWaTitle').value; let m = document.getElementById('newWaMsg').value; if(t && m) { if(!shopSettings.waTemplates) shopSettings.waTemplates =[]; shopSettings.waTemplates.push({title: t, msg: m}); document.getElementById('newWaTitle').value = ""; document.getElementById('newWaMsg').value = ""; syncSettingsToFirebase(); renderWaTemplatesInSettings(); } else { alert("يرجى ملء العنوان والنص."); } }
        function removeWaTemplate(index) { if(!verifyDelete()) return; if(confirm("حذف هذه الرسالة؟")) { shopSettings.waTemplates.splice(index, 1); syncSettingsToFirebase(); renderWaTemplatesInSettings(); } }

        // ----------------------------------------------------
        // 7. Order Cart & Logic
        // ----------------------------------------------------
        function populateNewOrderDropdowns() {
            let catSelect = document.getElementById('selCategory'); catSelect.innerHTML = '<option value="">اختر الصنف...</option>';
            let uniqueCategories =[...new Set(servicesData.map(s => s.category || 'عام'))]; uniqueCategories.forEach(c => catSelect.innerHTML += `<option value="${c}">${c}</option>`);
            let rentSelect = document.getElementById('selRental'); rentSelect.innerHTML = '<option value="">اختر لوازم الكراء...</option>'; rentalItemsData.forEach((r, i) => rentSelect.innerHTML += `<option value="${i}">${r.name} (${r.price} د.م)</option>`);
            filterServicesByCategory();
        }

        function filterServicesByCategory() {
            let cat = document.getElementById('selCategory').value; let srvSelect = document.getElementById('selService'); srvSelect.innerHTML = '<option value="">اختر الخدمة...</option>';
            if(cat !== "") { servicesData.forEach((s, i) => { let sCat = s.category || 'عام'; if(sCat === cat) { srvSelect.innerHTML += `<option value="${i}">${s.name} (${s.price} د.م)</option>`; } }); }
        }

        function addServiceToOrder() { let idx = document.getElementById('selService').value; let qty = parseInt(document.getElementById('qtyService').value) || 1; if (idx === "") return alert("الرجاء اختيار خدمة."); let srv = servicesData[idx]; cartServices.push({ name: srv.name, qty: qty, price: srv.price, subtotal: srv.price * qty }); document.getElementById('selService').value = ""; document.getElementById('qtyService').value = "1"; renderCartItems(); autoCalculateTotal(); }
        function addRentalToOrder() { let idx = document.getElementById('selRental').value; let qty = parseInt(document.getElementById('qtyRental').value) || 1; let date = document.getElementById('dateRentalReturn').value; if (idx === "") return alert("الرجاء اختيار لوازم."); if (!date) return alert("الرجاء تحديد تاريخ الاسترجاع."); let rent = rentalItemsData[idx]; cartRentals.push({ name: rent.name, qty: qty, price: rent.price, returnDate: date, subtotal: rent.price * qty, status: 'في عهدة الزبون' }); document.getElementById('selRental').value = ""; document.getElementById('qtyRental').value = "1"; document.getElementById('dateRentalReturn').value = ""; renderCartItems(); autoCalculateTotal(); }
        function removeCartService(idx) { cartServices.splice(idx, 1); renderCartItems(); autoCalculateTotal(); } function removeCartRental(idx) { cartRentals.splice(idx, 1); renderCartItems(); autoCalculateTotal(); }
        
        function renderCartItems() { 
            let srvList = document.getElementById('cartServicesList'); srvList.innerHTML = ""; cartServices.forEach((s, i) => { srvList.innerHTML += `<div class="cart-item"><div>${s.name} <span style="font-size:12px;">(العدد: ${s.qty})</span> - <b>${s.subtotal} د.م</b></div><button type="button" onclick="removeCartService(${i})">✖</button></div>`; }); 
            let rentList = document.getElementById('cartRentalsList'); rentList.innerHTML = ""; cartRentals.forEach((r, i) => { rentList.innerHTML += `<div class="cart-item" style="border-color:#c7d2fe;"><div>${r.name} <span style="font-size:12px;">(العدد: ${r.qty})</span> - <b>${r.subtotal} د.م</b><br><span style="color:var(--danger); font-size:12px;">📅 إرجاع: ${r.returnDate}</span></div><button type="button" onclick="removeCartRental(${i})">✖</button></div>`; }); 
        }
        function autoCalculateTotal() { let sum = cartServices.reduce((a, b) => a + b.subtotal, 0) + cartRentals.reduce((a, b) => a + b.subtotal, 0); document.getElementById('inpTotal').value = sum; calculateRemaining(); }
        function calculateRemaining() { let total = parseFloat(document.getElementById('inpTotal').value) || 0; let deposit = parseFloat(document.getElementById('inpDeposit').value) || 0; document.getElementById('inpRemaining').value = (total - deposit) < 0 ? 0 : (total - deposit); }

        function openNewOrderModal() { 
            isEditing = false; editingOrderId = null; document.getElementById('orderModalTitle').innerHTML = `إضافة طلب جديد <span class="order-id-badge" id="displayOrderId">#ORD-${Math.floor(1000 + Math.random() * 9000)}</span>`; 
            document.getElementById('inpDate').value = getTodayIsoString(); document.getElementById('inpName').value = ''; document.getElementById('inpPhone').value = ''; document.getElementById('inpNotes').value = ''; document.getElementById('inpTotal').value = ''; document.getElementById('inpDeposit').value = ''; document.getElementById('inpRemaining').value = '';
            cartServices = []; cartRentals =[]; renderCartItems(); populateNewOrderDropdowns(); document.getElementById('btnSaveOrderReal').innerText = "حفظ الطلب ✓"; openModal('orderModal'); 
        }

        function saveNewOrder() {
            let name = document.getElementById('inpName').value; let phone = document.getElementById('inpPhone').value; let date = document.getElementById('inpDate').value; let notes = document.getElementById('inpNotes').value; let orderId = document.getElementById('displayOrderId').textContent;
            if (!name || !phone || !date) return alert("يرجى ملء الاسم، الهاتف، وموعد التسليم."); if (cartServices.length === 0 && cartRentals.length === 0) return alert("أضف خدمة واحدة على الأقل.");
            document.getElementById('btnSaveOrderReal').innerText = "جاري الحفظ ☁️...";
            
            let regDate = isEditing ? ordersData.find(x=>x.id==orderId).registerDate : getTodayIsoString();
            let currentStatus = isEditing ? ordersData.find(x=>x.id==orderId).status : 'قيد الانتظار';

            let newOrder = { id: orderId, name: name, phone: phone, notes: notes, types:[...cartServices], rentals:[...cartRentals], date: date, registerDate: regDate, total: document.getElementById('inpTotal').value || 0, deposit: document.getElementById('inpDeposit').value || 0, remaining: document.getElementById('inpRemaining').value || 0, status: currentStatus };
            
            if(isEditing) { let idx = ordersData.findIndex(x=>x.id == orderId); if(idx > -1) ordersData[idx] = newOrder; } 
            else { ordersData.push(newOrder); }
            
            updateDashboardStats(); renderDualCalendars(currentMonth, currentYear);
            if(document.getElementById('trackingModal').classList.contains('active')) renderTrackingList();
            if(document.getElementById('ordersListModal').classList.contains('active')) refreshCurrentListModal();
            
            closeModal('orderModal'); currentSavedOrderId = orderId; 
            if(!isEditing) { document.getElementById('successOrderId').textContent = orderId; openModal('successModal'); }
            
            db.collection("orders").doc(orderId).set(newOrder).catch(err => { console.error(err); });
        }

        function editOrder(orderId) {
            let o = ordersData.find(x => x.id === orderId); if(!o) return;
            if(!verifyDelete()) return alert("كلمة مرور التعديل خاطئة!");
            isEditing = true; editingOrderId = orderId;
            document.getElementById('orderModalTitle').innerHTML = `تعديل الطلب <span class="order-id-badge" id="displayOrderId">${orderId}</span>`;
            document.getElementById('inpName').value = o.name || ''; document.getElementById('inpPhone').value = o.phone || ''; document.getElementById('inpDate').value = o.date || ''; document.getElementById('inpNotes').value = o.notes || '';
            cartServices = JSON.parse(JSON.stringify(o.types ||[])); cartRentals = JSON.parse(JSON.stringify(o.rentals ||[]));
            document.getElementById('inpTotal').value = o.total || 0; document.getElementById('inpDeposit').value = o.deposit || 0; document.getElementById('inpRemaining').value = o.remaining || 0;
            populateNewOrderDropdowns(); renderCartItems();
            document.getElementById('btnSaveOrderReal').innerText = "تحديث الطلب ✓";
            closeModal('ordersListModal'); closeModal('trackingModal'); openModal('orderModal');
        }

        function deleteOrder(orderId) {
            if(!verifyDelete()) return alert("كلمة مرور الحذف خاطئة!");
            if(confirm("هل أنت متأكد من حذف هذا الطلب نهائياً من النظام والسحابة؟")) {
                db.collection("orders").doc(orderId).delete().then(()=>{
                    ordersData = ordersData.filter(x => x.id !== orderId); updateDashboardStats(); renderDualCalendars(currentMonth, currentYear);
                    if(document.getElementById('ordersListModal').classList.contains('active')) refreshCurrentListModal();
                    if(document.getElementById('trackingModal').classList.contains('active')) renderTrackingList();
                    alert("تم الحذف بنجاح!");
                });
            }
        }

        // ----------------------------------------------------
        // 8. Order Tracking & Searching
        // ----------------------------------------------------
        function openTrackingModal() { renderTrackingList(); openModal('trackingModal'); }
        
        function calculateDaysDifference(targetDateStr) { 
            if (!targetDateStr) return 0;
            const today = new Date(); today.setHours(0,0,0,0); const target = new Date(targetDateStr); target.setHours(0,0,0,0); 
            return Math.ceil((target - today) / (1000 * 60 * 60 * 24)); 
        }
        
        function renderTrackingList() {
            const list = document.getElementById('trackingListContainer'); list.innerHTML = "";
            let tracked = ordersData.filter(o => o.status !== 'تم التسليم').map(o => { return { ...o, daysRemaining: calculateDaysDifference(o.date) }; }); tracked.sort((a, b) => a.daysRemaining - b.daysRemaining);
            if(tracked.length === 0) return list.innerHTML = "<div class='empty-day-msg'>لا توجد طلبيات قيد الانتظار.</div>";
            tracked.forEach(o => {
                let colorClass = o.daysRemaining < 5 ? "track-red" : (o.daysRemaining < 10 ? "track-orange" : "track-green");
                let statusText = o.daysRemaining < 0 ? `متأخر بـ ${Math.abs(o.daysRemaining)} أيام ⚠️` : (o.daysRemaining === 0 ? "التسليم اليوم! 🔴" : `متبقي ${o.daysRemaining} أيام`);
                let itemsStr =[]; if(o.types && o.types.length > 0) itemsStr.push(`🖨️ طباعة: ` + o.types.map(t=>`${t.name}(x${t.qty})`).join(' + ')); if(o.rentals && o.rentals.length > 0) itemsStr.push(`📦 كراء: ` + o.rentals.map(r=>`${r.name}(x${r.qty})`).join(' + '));
                list.innerHTML += `<div class="tracking-card ${colorClass}"><div class="tracking-info"><h4>${o.name} <span class="order-id-badge" style="font-size:11px;">${o.id}</span></h4><p>📅 تاريخ التسليم: <b>${o.date}</b></p><p style="font-size:13px;">${itemsStr.join(' <br> ')}</p></div><div class="tracking-status"><div class="days-badge">${statusText}</div><button class="btn-print-small" onclick="closeModal('trackingModal'); openDayDetails('${o.date}');">👁️ عرض التفاصيل</button></div></div>`;
            });
        }

        function searchOrders() {
            let q = document.getElementById('searchInput').value.trim().toLowerCase();
            if(q === "") return;
            currentViewContext = { type: 'search', value: q };
            document.getElementById('ordersListTitle').innerHTML = `🔍 نتائج البحث عن: <b style="color:var(--primary)">${q}</b>`;
            document.getElementById('btnDaySpecificAdd').style.display = 'none';
            let listContainer = document.getElementById('ordersListContainer'); listContainer.innerHTML = '';
            
            let filtered = ordersData.filter(o => o.phone.includes(q) || o.id.toLowerCase().includes(q) || o.name.toLowerCase().includes(q));
            if(filtered.length === 0) { listContainer.innerHTML = '<div class="empty-day-msg">لم يتم العثور على أي طلب.</div>'; } 
            else { filtered.sort((a, b) => new Date(b.registerDate) - new Date(a.registerDate)); filtered.forEach(o => listContainer.innerHTML += generateOrderCardHTML(o)); }
            openModal('ordersListModal');
        }

        function refreshCurrentListModal() { 
            if(currentViewContext.type === 'date') openDayDetails(currentViewContext.value); 
            else if(currentViewContext.type === 'status') openListModal(currentViewContext.value); 
            else if(currentViewContext.type === 'search') searchOrders(); 
        }

        function openDayDetails(dateString) { currentViewContext = { type: 'date', value: dateString }; document.getElementById('ordersListTitle').innerHTML = `📅 تسليم/إرجاع يوم: <b dir="ltr">${dateString}</b>`; let listContainer = document.getElementById('ordersListContainer'); listContainer.innerHTML = ''; let dayOrders = ordersData.filter(o => o.date === dateString || (o.rentals && o.rentals.some(r => r.returnDate === dateString))); if (dayOrders.length === 0) listContainer.innerHTML = '<div class="empty-day-msg">لا توجد طلبات هنا.</div>'; else dayOrders.forEach(o => listContainer.innerHTML += generateOrderCardHTML(o)); document.getElementById('btnDaySpecificAdd').style.display = 'flex'; openModal('ordersListModal'); }
        function openNewOrderWithDate() { closeModal('ordersListModal'); isEditing = false; editingOrderId = null; document.getElementById('displayOrderId').textContent = `ORD-${Math.floor(1000 + Math.random() * 9000)}`; document.getElementById('inpDate').value = currentViewContext.value; document.getElementById('inpName').value=''; document.getElementById('inpPhone').value=''; document.getElementById('inpNotes').value=''; document.getElementById('inpTotal').value=''; document.getElementById('inpDeposit').value=''; document.getElementById('inpRemaining').value=''; cartServices =[]; cartRentals =[]; renderCartItems(); populateNewOrderDropdowns(); document.getElementById('btnSaveOrderReal').innerText = "حفظ الطلب ✓"; document.getElementById('orderModalTitle').innerHTML = `إضافة طلب جديد <span class="order-id-badge" id="displayOrderId">#ORD-${Math.floor(1000 + Math.random() * 9000)}</span>`; openModal('orderModal'); }
        
        function openListModal(filterType) { currentViewContext = { type: 'status', value: filterType }; let title = filterType === 'pending' ? '⏳ الطلبات قيد الانتظار والإنجاز' : (filterType === 'delivered' ? '📦 الطلبات المُسلّمة' : '📦 لوازم في الكراء'); document.getElementById('ordersListTitle').innerHTML = title; document.getElementById('btnDaySpecificAdd').style.display = 'none'; let listContainer = document.getElementById('ordersListContainer'); listContainer.innerHTML = ''; let filteredOrders =[]; if(filterType === 'pending') filteredOrders = ordersData.filter(o => o.status !== 'تم التسليم'); else if (filterType === 'delivered') filteredOrders = ordersData.filter(o => o.status === 'تم التسليم'); else if (filterType === 'rentals') filteredOrders = ordersData.filter(o => o.rentals && o.rentals.some(r => r.status === 'في عهدة الزبون')); if (filteredOrders.length === 0) listContainer.innerHTML = '<div class="empty-day-msg">لا توجد طلبات في هذا القسم.</div>'; else { filteredOrders.sort((a, b) => new Date(a.date) - new Date(b.date)); filteredOrders.forEach(o => listContainer.innerHTML += generateOrderCardHTML(o)); } openModal('ordersListModal'); }

        function updateDashboardStats() { let pending = ordersData.filter(o => o.status !== 'تم التسليم').length; let delivered = ordersData.filter(o => o.status === 'تم التسليم').length; let rented = ordersData.filter(o => o.rentals && o.rentals.some(r => r.status === 'في عهدة الزبون')).length; document.getElementById('statsPending').textContent = `${pending} طلب`; document.getElementById('statsDelivered').textContent = `${delivered} طلب`; document.getElementById('statsRented').textContent = `${rented} طلب`; }

        // --- كروت الطلبات وقوائم التشغيل ---
        function generateOrderCardHTML(o) {
            let cardBg = o.status === 'تم التسليم' ? 'background-color: rgba(255,255,255,0.4); backdrop-filter: blur(5px); opacity: 0.8;' : '';
            let whatsappBtnHTML = o.status === 'جاهز' ? `<button class="btn-whatsapp" onclick="openWaSelectModal('${o.id}')">واتس آب 💬</button>` : `<button class="btn-whatsapp" style="background:#e2e8f0; color:var(--text-muted); box-shadow:none;" onclick="openWaSelectModal('${o.id}')">واتس آب 💬</button>`;
            let typesHTML = o.types && o.types.length > 0 ? `<p>🖨️ ${o.types.map(t => `${t.name} (x${t.qty})`).join(' + ')}</p>` : '';
            let rentalHTML = '';
            if(o.rentals && o.rentals.length > 0) {
                let rentDetails = o.rentals.map((r, i) => {
                    let actionBtn = r.status !== 'تم الإرجاع' ? `<button style="background:white; border:1px solid var(--indigo); color:var(--indigo); padding:4px 10px; border-radius:6px; font-size:12px; cursor:pointer;" onclick="markRentalItemReturned('${o.id}', ${i})">تأكيد الإرجاع ✓</button>` : '';
                    return `<div style="border-bottom: 1px dashed rgba(199, 210, 254, 0.5); padding: 5px 0;"><p style="margin:0; font-size:14px; font-weight:bold;">📦 ${r.name} (العدد: ${r.qty})</p><div style="display:flex; justify-content:space-between; align-items:center; margin-top:5px;"><span style="font-size:12px;">إرجاع: <b>${r.returnDate}</b> | الحالة: <b>${r.status}</b></span>${actionBtn}</div></div>`;
                }).join('');
                rentalHTML = `<div style="background-color: rgba(238, 242, 255, 0.6); padding: 10px; border-radius: 8px; border: 1px solid rgba(199, 210, 254, 0.8); margin-top: 10px;">${rentDetails}</div>`;
            }
            
            let actionButtons = `
                <div style="display:flex; gap:5px; margin-top:10px;">
                    <button class="btn-print-small" onclick="printReceipt('${o.id}')">🖨️ طباعة</button>
                    <button class="btn-print-small" style="color:var(--primary); border-color:var(--primary);" onclick="editOrder('${o.id}')">✏️ تعديل</button>
                    <button class="btn-print-small" style="color:var(--danger); border-color:var(--danger);" onclick="deleteOrder('${o.id}')">🗑️ حذف</button>
                </div>
            `;

            return `
                <div class="day-order-card" style="${cardBg}">
                    <div class="order-top-row">
                        <div class="day-order-info" style="flex: 1;">
                            <h4>${o.name} <span class="order-id-badge" style="font-size:11px; margin-right:5px;">${o.id}</span></h4>
                            ${typesHTML}
                            <p>📞 <span dir="ltr">${o.phone}</span></p>
                            <p style="font-size:12px; color:var(--primary); margin-top:5px; font-weight:bold;">📅 موعد التسليم: ${o.date}</p>
                            ${rentalHTML}
                        </div>
                        <div style="display:flex; flex-direction:column; gap:10px; align-items:flex-end; margin-right: 15px;">
                            <div class="day-order-price"><div class="total">${o.total} درهم</div><div class="rem">الباقي: ${o.remaining} درهم</div></div>
                            ${actionButtons}
                        </div>
                    </div>
                    <div class="status-bar">
                        <div>
                            <label style="font-size: 13px; font-weight: bold; color: var(--text-muted);">الحالة: </label>
                            <select class="status-select" onchange="changeOrderStatus('${o.id}', this.value)">
                                <option value="قيد الانتظار" ${o.status === 'قيد الانتظار' ? 'selected' : ''}>⏳ قيد الانتظار</option>
                                <option value="قيد الإنجاز" ${o.status === 'قيد الإنجاز' ? 'selected' : ''}>⚙️ قيد الإنجاز</option>
                                <option value="جاهز" ${o.status === 'جاهز' ? 'selected' : ''}>✅ جاهز</option>
                                <option value="تم التسليم" ${o.status === 'تم التسليم' ? 'selected' : ''}>📦 تم التسليم</option>
                            </select>
                        </div>
                        ${whatsappBtnHTML}
                    </div>
                </div>
            `;
        }

        function changeOrderStatus(orderId, newStatus) { db.collection("orders").doc(orderId).update({ status: newStatus }).catch(err => console.error(err)); }
        function markRentalItemReturned(orderId, rentalIndex) { let order = ordersData.find(o => o.id === orderId); if(order && order.rentals[rentalIndex]) { order.rentals[rentalIndex].status = 'تم الإرجاع'; db.collection("orders").doc(orderId).update({ rentals: order.rentals }); } }
        
        // --- WhatsApp Logic ---
        function openWaSelectModal(orderId) {
            currentWaOrderId = orderId;
            let container = document.getElementById('waSelectOptions'); container.innerHTML = "";
            (shopSettings.waTemplates ||[]).forEach((tpl, i) => {
                container.innerHTML += `<button class="btn-whatsapp" style="width:100%; justify-content:center; margin-bottom:5px;" onclick="sendWhatsApp(${i})">${tpl.title}</button>`;
            });
            if(shopSettings.waTemplates && shopSettings.waTemplates.length === 0) container.innerHTML = "<p>لا توجد رسائل جاهزة، أضفها من الإعدادات.</p>";
            openModal('waSelectModal');
        }

        function sendWhatsApp(templateIndex) {
            let order = ordersData.find(o => o.id === currentWaOrderId);
            let tpl = shopSettings.waTemplates[templateIndex];
            if(order && tpl) {
                let msg = tpl.msg.replace(/\[الاسم\]/g, order.name).replace(/\[رقم_الطلب\]/g, order.id);
                let phone = order.phone.startsWith('0') ? '212' + order.phone.substring(1) : order.phone;
                window.open(`https://wa.me/${phone}?text=${encodeURIComponent(msg)}`, '_blank');
                closeModal('waSelectModal');
            }
        }

        // =========================================================
        // 🖨️ الطباعة الصافية لمقاس A5 (مع الملاحظات والأثمنة و QR)
        // =========================================================
        function printReceipt(orderId) {
            let order = ordersData.find(o => o.id === orderId); if(!order) return;
            if(shopSettings.logo) { document.getElementById('r-shopLogo').src = shopSettings.logo; document.getElementById('r-shopLogo').style.display = 'block'; document.getElementById('r-shopName').style.display = 'none'; } else { document.getElementById('r-shopLogo').style.display = 'none'; document.getElementById('r-shopName').style.display = 'block'; document.getElementById('r-shopName').textContent = shopSettings.name; }

            let d = new Date(order.registerDate); let dayName = daysArabic[d.getDay()] || "";
            document.getElementById('r-today').textContent = `${dayName}، ${order.registerDate}`;
            document.getElementById('r-tagline').textContent = shopSettings.tagline; document.getElementById('r-shopPhone').textContent = shopSettings.phone; document.getElementById('r-shopAddress').textContent = shopSettings.address;
            document.getElementById('r-id').textContent = order.id; document.getElementById('r-name').textContent = order.name; document.getElementById('r-phone').textContent = order.phone; document.getElementById('r-date').textContent = order.date;
            
            if(order.notes && order.notes.trim() !== "") { document.getElementById('r-notes-row').style.display = 'table-row'; document.getElementById('r-notes').textContent = order.notes; } else { document.getElementById('r-notes-row').style.display = 'none'; }

            let itemsHTML = '<table class="items-table"><tr><th style="text-align:right;">الخدمة</th><th style="text-align:center;">العدد</th><th style="text-align:left;">الثمن</th></tr>';
            if(order.types && order.types.length > 0) { order.types.forEach(t => { itemsHTML += `<tr><td style="font-weight:bold;">🖨️ ${t.name}</td><td style="text-align:center;">x${t.qty}</td><td style="text-align:left;">${t.subtotal} د.م</td></tr>`; }); }
            if(order.rentals && order.rentals.length > 0) { order.rentals.forEach(r => { itemsHTML += `<tr><td style="font-weight:bold;">📦 ${r.name} <br><span style="font-size:11px; color:#555;">إرجاع: ${r.returnDate}</span></td><td style="text-align:center;">x${r.qty}</td><td style="text-align:left;">${r.subtotal} د.م</td></tr>`; }); }
            itemsHTML += '</table>'; document.getElementById('r-items-list').innerHTML = itemsHTML;
            
            document.getElementById('r-total').textContent = order.total + " درهم"; document.getElementById('r-deposit').textContent = order.deposit + " درهم"; document.getElementById('r-remaining').textContent = order.remaining + " درهم";

            if(shopSettings.socialQr) { document.getElementById('r-socialQr').src = shopSettings.socialQr; document.getElementById('social-qr-container').style.display = 'block'; } else { document.getElementById('social-qr-container').style.display = 'none'; }

            let trackingUrl = ""; if (window.location.protocol === 'file:') trackingUrl = "https://mawa3id-mis-designe.web.app/?track=" + order.id; else trackingUrl = window.location.href.split('?')[0] + "?track=" + order.id;
            let qrContainer = document.getElementById('r-qrcode'); qrContainer.innerHTML = ""; 
            try { new QRCode(qrContainer, { text: trackingUrl, width: 60, height: 60, colorDark : "#000000", colorLight : "#ffffff", correctLevel : QRCode.CorrectLevel.L }); } catch(e) {}
            
            setTimeout(() => { window.print(); }, 500);
        }

        // --- التقويم ---
        const monthNames =["يناير", "فبراير", "مارس", "أبريل", "مايو", "يونيو", "يوليو", "أغسطس", "سبتمبر", "أكتوبر", "نوفمبر", "ديسمبر"]; let currentMonth = new Date().getMonth(); let currentYear = new Date().getFullYear();
        function renderGrid(month, year, gridId) { const grid = document.getElementById(gridId); grid.innerHTML = ""; let firstDay = new Date(year, month, 1).getDay(); let daysInMonth = new Date(year, month + 1, 0).getDate(); let today = new Date(); for (let i = 0; i < firstDay; i++) { let emptyDiv = document.createElement("div"); emptyDiv.classList.add("day", "empty"); grid.appendChild(emptyDiv); } for (let i = 1; i <= daysInMonth; i++) { let dayDiv = document.createElement("div"); dayDiv.classList.add("day"); dayDiv.setAttribute("data-day", i); let span = document.createElement("span"); span.classList.add("day-number"); span.textContent = i; dayDiv.appendChild(span); if (i === today.getDate() && month === today.getMonth() && year === today.getFullYear()) dayDiv.classList.add("today"); let currentCellDate = getIsoDate(year, month, i); let ordersInThisDay = ordersData.filter(o => (o.date === currentCellDate && o.status !== 'تم التسليم') || (o.rentals && o.rentals.some(r => r.returnDate === currentCellDate && r.status === 'في عهدة الزبون'))); if (ordersInThisDay.length > 0) { dayDiv.classList.add("has-orders"); let badge = document.createElement("div"); badge.classList.add("order-badge"); badge.textContent = ordersInThisDay.length; dayDiv.appendChild(badge); } dayDiv.addEventListener('click', () => { openDayDetails(currentCellDate); }); grid.appendChild(dayDiv); } }
        function renderDualCalendars(month1, year1) { let month2 = month1 + 1; let year2 = year1; if (month2 > 11) { month2 = 0; year2 = year1 + 1; } document.getElementById("month1Title").textContent = monthNames[month1]; document.getElementById("month2Title").textContent = monthNames[month2]; document.getElementById("masterYearDisplay").textContent = (year1 === year2) ? `سنة ${year1}` : `${year1} - ${year2}`; renderGrid(month1, year1, "daysGrid1"); renderGrid(month2, year2, "daysGrid2"); }
        document.getElementById("prevMonth").addEventListener("click", () => { currentMonth--; if (currentMonth < 0) { currentMonth = 11; currentYear--; } renderDualCalendars(currentMonth, currentYear); }); document.getElementById("nextMonth").addEventListener("click", () => { currentMonth++; if (currentMonth > 11) { currentMonth = 0; currentYear++; } renderDualCalendars(currentMonth, currentYear); });

        // تصدير واستيراد
        function exportDataToJSON() { const fullData = { orders: ordersData, settings: shopSettings, services: servicesData, rentals: rentalItemsData, categories: categoriesData }; const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(fullData)); const dl = document.createElement('a'); dl.setAttribute("href", dataStr); dl.setAttribute("download", `PrintShop_${new Date().toISOString().slice(0,10)}.json`); document.body.appendChild(dl); dl.click(); dl.remove(); }
        function importDataFromJSON(event) { const reader = new FileReader(); reader.onload = function(e) { try { const imp = JSON.parse(e.target.result); if(imp.orders) { imp.orders.forEach(o => db.collection("orders").doc(o.id).set(o)); } db.collection("config").doc("main").set({ shopSettings: imp.settings || shopSettings, servicesData: imp.services || servicesData, rentalItemsData: imp.rentals || rentalItemsData, categoriesData: imp.categories || categoriesData }); closeModal('settingsModal'); alert("تم المزامنة بنجاح!"); } catch(err) { alert("ملف غير صالح!"); } }; if(event.target.files[0]) reader.readAsText(event.target.files[0]); }
    </script>
</body>
</html>
