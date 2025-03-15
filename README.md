<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Arkadaş Buluşması Katılım Takibi</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
        h1 { color: #2c3e50; text-align: center; }
        .card { background-color: white; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); padding: 20px; margin-bottom: 20px; }
        .input-group { display: flex; margin-bottom: 15px; }
        input, select { flex: 1; padding: 10px; border: 1px solid #ddd; border-radius: 4px; font-size: 16px; }
        button { background-color: #3498db; color: white; border: none; border-radius: 4px; padding: 10px 15px; cursor: pointer; font-size: 16px; }
        button:hover { background-color: #2980b9; }
        .button-danger { background-color: #e74c3c; }
        .button-danger:hover { background-color: #c0392b; }
        table { width: 100%; border-collapse: collapse; }
        th, td { padding: 12px; text-align: left; border-bottom: 1px solid #ddd; }
        .status-evet { background-color: #2ecc71; color: white; padding: 2px 5px; border-radius: 3px; }
        .status-hayir { background-color: #e74c3c; color: white; padding: 2px 5px; border-radius: 3px; }
        .status-belki { background-color: #f39c12; color: white; padding: 2px 5px; border-radius: 3px; }
        .tabs { display: flex; margin-bottom: 20px; }
        .tab { padding: 10px; cursor: pointer; border: 1px solid #ddd; flex: 1; text-align: center; }
        .tab.active { background-color: #3498db; color: white; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        #exportArea { width: 100%; height: 100px; margin-top: 10px; padding: 10px; border: 1px solid #ddd; resize: vertical; }
    </style>
</head>
<body>
    <h1>Arkadaş Buluşması Katılım Takibi</h1>
    
    <div class="tabs">
        <div class="tab active" data-tab="event-details">Buluşma Detayları</div>
        <div class="tab" data-tab="participants">Katılımcılar</div>
        <div class="tab" data-tab="export">Dışa Aktar</div>
    </div>
    
    <div id="event-details" class="tab-content active">
        <div class="card">
            <h2>Buluşma Bilgileri</h2>
            <div class="input-group">
                <input type="text" id="eventName" placeholder="Etkinlik Adı">
            </div>
            <div class="input-group">
                <input type="text" id="eventLocation" placeholder="Konum">
            </div>
            <div class="input-group">
                <input type="date" id="eventDate">
                <input type="time" id="eventTime">
            </div>
            <div class="input-group">
                <input type="text" id="eventDescription" placeholder="Açıklama">
            </div>
            <button id="saveEventButton">Kaydet</button>
        </div>
        
        <div class="card">
            <h2>Etkinlik Detayları</h2>
            <div id="eventDetails">
                <p>Henüz etkinlik detayları girilmedi.</p>
            </div>
        </div>
    </div>
    
    <div id="participants" class="tab-content">
        <div class="card">
            <h2>Katılımcı Ekle</h2>
            <div class="input-group">
                <input type="text" id="participantName" placeholder="İsim">
                <input type="text" id="participantPhone" placeholder="Telefon (isteğe bağlı)">
            </div>
            <div class="input-group">
                <select id="participantBranch">
                    <option value="">Şube Seçin</option>
                    <option value="A">A</option>
                    <option value="B">B</option>
                    <option value="C">C</option>
                    <option value="D">D</option>
                </select>
                <input type="text" id="participantNote" placeholder="Not (isteğe bağlı)">
            </div>
            <div class="input-group">
                <select id="participantStatus">
                    <option value="evet">Katılacak</option>
                    <option value="hayir">Katılmayacak</option>
                    <option value="belki">Belki</option>
                </select>
            </div>
            <button id="addParticipantButton">Ekle</button>
        </div>
        
        <div class="card">
            <h2>Katılımcılar</h2>
            <div id="participantList">
                <table id="participantsTable">
                    <thead>
                        <tr>
                            <th>İsim</th>
                            <th>Telefon</th>
                            <th>Şube</th>
                            <th>Not</th>
                            <th>Durum</th>
                            <th>İşlemler</th>
                        </tr>
                    </thead>
                    <tbody>
                        <!-- Katılımcılar buraya JavaScript ile eklenecek -->
                    </tbody>
                </table>
                <p id="noParticipantsMessage">Henüz katılımcı eklenmedi.</p>
            </div>
        </div>
    </div>
    
    <div id="export" class="tab-content">
        <div class="card">
            <h2>Dışa Aktar</h2>
            <p>Aşağıdaki bilgileri kopyalayarak arkadaşlarınızla paylaşabilirsiniz:</p>
            <textarea id="exportArea" readonly></textarea>
            <div class="export-section">
                <button id="copyButton">Kopyala</button>
                <button id="clearAllButton" class="button-danger">Tüm Verileri Temizle</button>
            </div>
        </div>
    </div>
    
    <script>
        document.addEventListener('DOMContentLoaded', function() {
            // Değişkenler
            let participants = [];
            let eventInfo = {
                name: '',
                location: '',
                date: '',
                time: '',
                description: ''
            };
            
            // Yerel depolamadan verileri yükle
            loadFromLocalStorage();
            
            // DOM elementleri
            const tabs = document.querySelectorAll('.tab');
            const tabContents = document.querySelectorAll('.tab-content');
            const saveEventButton = document.getElementById('saveEventButton');
            const addParticipantButton = document.getElementById('addParticipantButton');
            const copyButton = document.getElementById('copyButton');
            const clearAllButton = document.getElementById('clearAllButton');
            const participantsTable = document.getElementById('participantsTable');
            const noParticipantsMessage = document.getElementById('noParticipantsMessage');
            
            // Tab kontrolü
            tabs.forEach(tab => {
                tab.addEventListener('click', function() {
                    const tabId = this.getAttribute('data-tab');
                    
                    // Tüm tabları ve içeriklerini deaktif et
                    tabs.forEach(t => t.classList.remove('active'));
                    tabContents.forEach(c => c.classList.remove('active'));
                    
                    // Seçilen tabı ve içeriğini aktif et
                    this.classList.add('active');
                    document.getElementById(tabId).classList.add('active');
                    
                    // Dışa aktar tabındaysak içeriği güncelle
                    if (tabId === 'export') {
                        updateExportArea();
                    }
                });
            });
            
            // Etkinlik bilgilerini kaydet
            saveEventButton.addEventListener('click', function() {
                eventInfo = {
                    name: document.getElementById('eventName').value,
                    location: document.getElementById('eventLocation').value,
                    date: document.getElementById('eventDate').value,
                    time: document.getElementById('eventTime').value,
                    description: document.getElementById('eventDescription').value
                };
                
                updateEventDetails();
                saveToLocalStorage();
                alert('Etkinlik bilgileri kaydedildi!');
            });
            
            // Katılımcı ekle
            addParticipantButton.addEventListener('click', function() {
                const name = document.getElementById('participantName').value.trim();
                const phone = document.getElementById('participantPhone').value.trim();
                const note = document.getElementById('participantNote').value.trim();
                const status = document.getElementById('participantStatus').value;
                const branch = document.getElementById('participantBranch').value;
                
                if (name === '') {
                    alert('Lütfen bir isim girin!');
                    return;
                }
                
                if (branch === '') {
                    alert('Lütfen bir şube seçin!');
                    return;
                }
                
                const participant = {
                    id: Date.now(),
                    name: name,
                    phone: phone,
                    branch: branch,
                    note: note,
                    status: status
                };
                
                participants.push(participant);
                updateParticipantList();
                saveToLocalStorage();
                
                // Inputları temizle
                document.getElementById('participantName').value = '';
                document.getElementById('participantPhone').value = '';
                document.getElementById('participantNote').value = '';
                document.getElementById('participantBranch').value = '';
                document.getElementById('participantStatus').value = 'evet';
            });
            
            // Kopyala butonu
            copyButton.addEventListener('click', function() {
                const exportArea = document.getElementById('exportArea');
                exportArea.select();
                document.execCommand('copy');
                alert('Bilgiler panoya kopyalandı!');
            });
            
            // Tüm verileri temizle
            clearAllButton.addEventListener('click', function() {
                if (confirm('Tüm verileri silmek istediğinizden emin misiniz?')) {
                    participants = [];
                    eventInfo = {
                        name: '',
                        location: '',
                        date: '',
                        time: '',
                        description: ''
                    };
                    
                    document.getElementById('eventName').value = '';
                    document.getElementById('eventLocation').value = '';
                    document.getElementById('eventDate').value = '';
                    document.getElementById('eventTime').value = '';
                    document.getElementById('eventDescription').value = '';
                    
                    updateEventDetails();
                    updateParticipantList();
                    saveToLocalStorage();
                }
            });
            
            // Katılımcı listesini güncelle
            function updateParticipantList() {
                const tbody = participantsTable.querySelector('tbody');
                tbody.innerHTML = '';
                
                if (participants.length === 0) {
                    noParticipantsMessage.style.display = 'block';
                    participantsTable.style.display = 'none';
                    return;
                }
                
                noParticipantsMessage.style.display = 'none';
                participantsTable.style.display = 'table';
                
                // Katılımcıları alfabetik sırala
                const sortedParticipants = [...participants].sort((a, b) => 
                    a.name.localeCompare(b.name, 'tr')
                );
                
                sortedParticipants.forEach(participant => {
                    const row = document.createElement('tr');
                    
                    // İsim
                    const nameCell = document.createElement('td');
                    nameCell.textContent = participant.name;
                    row.appendChild(nameCell);
                    
                    // Telefon
                    const phoneCell = document.createElement('td');
                    phoneCell.textContent = participant.phone || '-';
                    row.appendChild(phoneCell);
                    
                    // Şube
                    const branchCell = document.createElement('td');
                    branchCell.textContent = participant.branch || '-';
                    row.appendChild(branchCell);
                    
                    // Not
                    const noteCell = document.createElement('td');
                    noteCell.textContent = participant.note || '-';
                    row.appendChild(noteCell);
                    
                    // Durum
                    const statusCell = document.createElement('td');
                    const statusSpan = document.createElement('span');
                    statusSpan.classList.add(`status-${participant.status}`);
                    
                    if (participant.status === 'evet') {
                        statusSpan.textContent = 'Katılacak';
                    } else if (participant.status === 'hayir') {
                        statusSpan.textContent = 'Katılmayacak';
                    } else {
                        statusSpan.textContent = 'Belki';
                    }
                    
                    statusCell.appendChild(statusSpan);
                    row.appendChild(statusCell);
                    
                    // İşlemler
                    const actionsCell = document.createElement('td');
                    
                    // Durum değiştir butonu
                    const changeStatusButton = document.createElement('button');
                    changeStatusButton.textContent = 'Durum';
                    changeStatusButton.addEventListener('click', function() {
                        changeParticipantStatus(participant.id);
                    });
                    
                    // Sil butonu
                    const deleteButton = document.createElement('button');
                    deleteButton.textContent = 'Sil';
                    deleteButton.classList.add('button-danger');
                    deleteButton.addEventListener('click', function() {
                        deleteParticipant(participant.id);
                    });
                    
                    actionsCell.appendChild(changeStatusButton);
                    actionsCell.appendChild(deleteButton);
                    row.appendChild(actionsCell);
                    
                    tbody.appendChild(row);
                });
            }
            
            // Etkinlik detaylarını güncelle
            function updateEventDetails() {
                const eventDetails = document.getElementById('eventDetails');
                
                if (!eventInfo.name && !eventInfo.location && !eventInfo.date) {
                    eventDetails.innerHTML = '<p>Henüz etkinlik detayları girilmedi.</p>';
                    return;
                }
                
                let html = '';
                
                if (eventInfo.name) {
                    html += `<p><strong>Etkinlik:</strong> ${eventInfo.name}</p>`;
                }
                
                if (eventInfo.location) {
                    html += `<p><strong>Konum:</strong> ${eventInfo.location}</p>`;
                }
                
                if (eventInfo.date) {
                    html += `<p><strong>Tarih:</strong> ${eventInfo.date}</p>`;
                }
                
                if (eventInfo.time) {
                    html += `<p><strong>Saat:</strong> ${eventInfo.time}</p>`;
                }
                
                if (eventInfo.description) {
                    html += `<p><strong>Açıklama:</strong> ${eventInfo.description}</p>`;
                }
                
                eventDetails.innerHTML = html;
            }
            
            // Dışa aktarım alanını güncelle
            function updateExportArea() {
                const exportArea = document.getElementById('exportArea');
                let text = '';
                
                if (eventInfo.name) {
                    text += `${eventInfo.name}\n`;
                } else {
                    text += 'Arkadaş Buluşması\n';
                }
                
                if (eventInfo.date) {
                    text += `Tarih: ${eventInfo.date}\n`;
                }
                
                if (eventInfo.time) {
                    text += `Saat: ${eventInfo.time}\n`;
                }
                
                if (eventInfo.location) {
                    text += `Konum: ${eventInfo.location}\n`;
                }
                
                if (eventInfo.description) {
                    text += `Not: ${eventInfo.description}\n`;
                }
                
                text += '\nKatılımcılar:\n';
                
                const coming = participants.filter(p => p.status === 'evet');
                const maybe = participants.filter(p => p.status === 'belki');
                const notComing = participants.filter(p => p.status === 'hayir');
                
                if (coming.length > 0) {
                    text += '\nKatılacaklar:\n';
                    // Alfabetik sırala
                    const sortedComing = [...coming].sort((a, b) => a.name.localeCompare(b.name, 'tr'));
                    sortedComing.forEach(p => {
                        text += `- ${p.name} (Şube: ${p.branch})${p.note ? ` (Not: ${p.note})` : ''}\n`;
                    });
                }
                
                if (maybe.length > 0) {
                    text += '\nBelki Katılacaklar:\n';
                    // Alfabetik sırala
                    const sortedMaybe = [...maybe].sort((a, b) => a.name.localeCompare(b.name, 'tr'));
                    sortedMaybe.forEach(p => {
                        text += `- ${p.name} (Şube: ${p.branch})${p.note ? ` (Not: ${p.note})` : ''}\n`;
                    });
                }
                
                if (notComing.length > 0) {
                    text += '\nKatılmayacaklar:\n';
                    // Alfabetik sırala
                    const sortedNotComing = [...notComing].sort((a, b) => a.name.localeCompare(b.name, 'tr'));
                    sortedNotComing.forEach(p => {
                        text += `- ${p.name} (Şube: ${p.branch})${p.note ? ` (Not: ${p.note})` : ''}\n`;
                    });
                }
                
                exportArea.value = text;
            }
            
            // Katılımcının durumunu değiştir
            function changeParticipantStatus(id) {
                const participant = participants.find(p => p.id === id);
                
                if (participant) {
                    // Durumları döndür: evet -> belki -> hayır -> evet
                    if (participant.status === 'evet') {
                        participant.status = 'belki';
                    } else if (participant.status === 'belki') {
                        participant.status = 'hayir';
                    } else {
                        participant.status = 'evet';
                    }
                    
                    updateParticipantList();
                    saveToLocalStorage();
                }
            }
            
            // Katılımcıyı sil
            function deleteParticipant(id) {
                if (confirm('Bu katılımcıyı silmek istediğinizden emin misiniz?')) {
                    participants = participants.filter(p => p.id !== id);
                    updateParticipantList();
                    saveToLocalStorage();
                }
            }
            
            // Yerel depolamaya kaydet
            function saveToLocalStorage() {
                localStorage.setItem('eventInfo', JSON.stringify(eventInfo));
                localStorage.setItem('participants', JSON.stringify(participants));
            }
            
            // Yerel depolamadan yükle
            function loadFromLocalStorage() {
                const savedEventInfo = localStorage.getItem('eventInfo');
                const savedParticipants = localStorage.getItem('participants');
                
                if (savedEventInfo) {
                    eventInfo = JSON.parse(savedEventInfo);
                    document.getElementById('eventName').value = eventInfo.name || '';
                    document.getElementById('eventLocation').value = eventInfo.location || '';
                    document.getElementById('eventDate').value = eventInfo.date || '';
                    document.getElementById('eventTime').value = eventInfo.time || '';
                    document.getElementById('eventDescription').value = eventInfo.description || '';
                    updateEventDetails();
                }
                
                if (savedParticipants) {
                    participants = JSON.parse(savedParticipants);
                    updateParticipantList();
                }
            }
            
            // Başlangıçta etkinlik detayları ve katılımcı listesini güncelle
            updateEventDetails();
            updateParticipantList();
        });
    </script>
</body>
</html>
