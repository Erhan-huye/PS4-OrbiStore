function renderDlcsEditor() {
            const container = document.getElementById('dlcsContainer');
            if (currentEditingDlcs.length === 0) {
                container.innerHTML = `<p class="text-xs text-slate-500 text-center py-4">Henüz DLC eklenmedi.</p>`;
                return;
            }
            container.innerHTML = currentEditingDlcs.map((dlc, idx) => `
                <div class="bg-slate-900 p-3 rounded-xl border border-slate-800 space-y-3">
                    <div class="flex justify-between items-center">
                        <span class="font-bold text-purple-300">#${idx+1} DLC Paketi</span>
                        <button type="button" onclick="removeDlcRow(${idx})" class="nav-focusable text-red-400 hover:text-red-300 text-xs"><i class="fa-solid fa-trash mr-1"></i>Kaldır</button>
                    </div>
                    <div class="grid grid-cols-1 sm:grid-cols-3 gap-2">
                        <input type="text" placeholder="DLC Adı" value="${dlc.name || ''}" oninput="currentEditingDlcs[${idx}].name=this.value" class="bg-slate-950 border border-slate-700 rounded-lg px-2.5 py-1 text-slate-200">
                        <input type="text" placeholder="Kategori (Örn: Kostüm, Genişleme)" value="${dlc.category || ''}" oninput="currentEditingDlcs[${idx}].category=this.value" class="bg-slate-950 border border-slate-700 rounded-lg px-2.5 py-1 text-slate-200">
                        <input type="text" placeholder="Boyut (Örn: 350 MB)" value="${dlc.size || ''}" oninput="currentEditingDlcs[${idx}].size=this.value" class="bg-slate-950 border border-slate-700 rounded-lg px-2.5 py-1 text-slate-200 font-mono">
                    </div>
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">
                        <input type="text" placeholder="Bölge / CUSA" value="${dlc.region || ''}" oninput="currentEditingDlcs[${idx}].region=this.value" class="bg-slate-950 border border-slate-700 rounded-lg px-2.5 py-1 text-slate-200 font-mono">
                        <input type="text" placeholder="İndirme URL (.pkg)" value="${dlc.url || ''}" oninput="currentEditingDlcs[${idx}].url=this.value" class="bg-slate-950 border border-slate-700 rounded-lg px-2.5 py-1 text-slate-200 font-mono">
                    </div>
                    <input type="text" placeholder="Notlar" value="${dlc.notes || ''}" oninput="currentEditingDlcs[${idx}].notes=this.value" class="w-full bg-slate-950 border border-slate-700 rounded-lg px-2.5 py-1 text-slate-200">
                </div>
            `).join('');
            refreshFocusables();
        }

        function saveGameForm(event) {
            event.preventDefault();
            const idVal = document.getElementById('editGameId').value;
            const title = document.getElementById('editTitle').value.trim();
            const cusa = document.getElementById('editCusa').value.trim();
            const category = document.getElementById('editCategory').value;
            const year = document.getElementById('editYear').value.trim();
            const size = document.getElementById('editSize').value.trim();
            const developer = document.getElementById('editDeveloper').value.trim();
            const region = document.getElementById('editRegion').value.trim();
            const version = document.getElementById('editVersion').value.trim();
            const image = document.getElementById('editImage').value.trim();
            const banner = document.getElementById('editBanner').value.trim();
            const url = document.getElementById('editUrl').value.trim();
            const description = document.getElementById('editDesc').value.trim();
            const tips = document.getElementById('editTips').value.trim();

            if (!title || !cusa) {
                showToast("Lütfen zorunlu alanları (Oyun Başlığı ve CUSA) doldurun.", "info");
                return;
            }

            if (idVal && idVal !== "") {
                const numericId = parseInt(idVal);
                const index = itemsDB.findIndex(i => i.id === numericId);
                if (index !== -1) {
                    itemsDB[index] = {
                        ...itemsDB[index],
                        title,
                        cusa,
                        category,
                        year,
                        size,
                        developer,
                        region,
                        version,
                        image,
                        banner,
                        url,
                        description,
                        tips,
                        updates: JSON.parse(JSON.stringify(currentEditingUpdates)),
                        dlcs: JSON.parse(JSON.stringify(currentEditingDlcs)),
                        tags: [category, "popular"]
                    };
                    showToast(`"${title}" başarıyla güncellendi.`, "success");
                }
            } else {
                const newGame = {
                    id: Date.now(),
                    title,
                    cusa,
                    category,
                    year,
                    size,
                    developer,
                    region,
                    version,
                    image,
                    banner,
                    url,
                    description,
                    tips,
                    updates: JSON.parse(JSON.stringify(currentEditingUpdates)),
                    dlcs: JSON.parse(JSON.stringify(currentEditingDlcs)),
                    tags: [category, "popular"]
                };
                itemsDB.push(newGame);
                showToast(`"${title}" kataloğa eklendi.`, "success");
            }

            localStorage.setItem('orbistore_catalog_v6', JSON.stringify(itemsDB));
            closeGameEditor();
            renderAdminTable();
            renderRows();
            initHeroShowcase();
        }

        function deleteGame(id) {
            if (confirm("Bu oyunu katalogdan silmek istediğinize emin misiniz?")) {
                itemsDB = itemsDB.filter(i => i.id !== id);
                localStorage.setItem('orbistore_catalog_v6', JSON.stringify(itemsDB));
                renderAdminTable();
                renderRows();
                initHeroShowcase();
                showToast("Oyun başarıyla silindi.", "info");
            }
        }

        function toggleQueueModal() {
            const modal = document.getElementById('queueModal');
            if (modal.classList.contains('hidden')) {
                modal.classList.remove('hidden');
                modal.classList.add('flex');
                renderQueueList();
            } else {
                modal.classList.remove('flex');
                modal.classList.add('hidden');
                refreshFocusables();
            }
        }

        function addToQueue(title, url) {
            downloadQueue.push({ id: Date.now(), title, url, status: 'İndiriliyor', progress: 45 });
            updateQueueBadge();
            showToast(`${title} indirme kuyruğuna eklendi.`, 'success');
            window.open(url, '_blank');
            closeDetailModal();
            setTimeout(() => {
                if(downloadQueue.length > 0) {
                    downloadQueue[0].status = 'Tamamlandı';
                    downloadQueue[0].progress = 100;
                    updateQueueBadge();
                    if (!document.getElementById('queueModal').classList.contains('hidden')) {
                        renderQueueList();
                    }
                }
            }, 3000);
        }

        function updateQueueBadge() {
            const badge = document.getElementById('queueBadge');
            const count = downloadQueue.filter(q => q.status === 'İndiriliyor').length;
            if (count > 0) {
                badge.textContent = count;
                badge.classList.remove('hidden');
            } else {
                badge.classList.add('hidden');
            }
        }

        function renderQueueList() {
            const list = document.getElementById('queueList');
            if (downloadQueue.length === 0) {
                list.innerHTML = `<p class="text-xs text-slate-500 text-center py-6">Kuyrukta aktif indirme yok.</p>`;
                return;
            }
            list.innerHTML = downloadQueue.map(q => `
                <div class="bg-slate-900/60 p-3 rounded-xl border border-slate-800 space-y-2">
                    <div class="flex justify-between items-center text-xs">
                        <span class="font-bold text-slate-200">${q.title}</span>
                        <span class="${q.status === 'Tamamlandı' ? 'text-ps4-goldhen' : 'text-cyan-400'} font-semibold">${q.status}</span>
                    </div>
                    <div class="w-full bg-slate-800 h-1.5 rounded-full overflow-hidden">
                        <div class="bg-ps4-accent h-full" style="width: ${q.progress}%"></div>
                    </div>
                </div>
            `).join('');
            refreshFocusables();
        }

        function clearCompletedQueue() {
            downloadQueue = downloadQueue.filter(q => q.status !== 'Tamamlandı');
            renderQueueList();
            updateQueueBadge();
        }

        function showToast(message, type = 'success') {
            const container = document.getElementById('toastContainer');
            const toast = document.createElement('div');
            toast.className = `cinematic-fade pointer-events-auto bg-slate-900/95 border border-slate-700/80 backdrop-blur-md px-4 py-3 rounded-xl shadow-2xl flex items-center space-x-3 text-xs`;
            
            const icon = type === 'success' ? '<i class="fa-solid fa-circle-check text-ps4-goldhen text-base"></i>' : '<i class="fa-solid fa-circle-info text-cyan-400 text-base"></i>';
            toast.innerHTML = `${icon}<div><div class="font-bold text-slate-100">GoldHEN Bildirimi</div><div class="text-slate-400">${message}</div></div>`;
            container.appendChild(toast);

            setTimeout(() => {
                toast.style.opacity = '0';
                toast.style.transition = 'opacity 0.3s ease';
                setTimeout(() => toast.remove(), 300);
            }, 3500);
        }

        function updateGamepadStatusUI() {
            const statusEl = document.getElementById('gamepadStatus');
            const dotEl = document.getElementById('gamepadDot');
            if (!statusEl || !dotEl) return;

            let connectedGamepads = [];
            if (navigator.getGamepads) {
                const gps = navigator.getGamepads();
                for (let i = 0; i < gps.length; i++) {
                    if (gps[i]) connectedGamepads.push(gps[i]);
                }
            }

            if (connectedGamepads.length > 0) {
                gamepadConnected = true;
                const gpName = connectedGamepads[0].id.toLowerCase();
                if (gpName.includes('sony') || gpName.includes('dualsense') || gpName.includes('dualshock') || gpName.includes('wireless controller')) {
                    statusEl.textContent = "DS4 Bağlı";
                    dotEl.className = "inline-block w-2 h-2 rounded-full bg-ps4-goldhen animate-pulse";
                } else {
                    statusEl.textContent = "Gamepad Bağlı";
                    dotEl.className = "inline-block w-2 h-2 rounded-full bg-cyan-400 animate-pulse";
                }
            } else {
                gamepadConnected = false;
                statusEl.textContent = "DS4 Bağlı Değil";
                dotEl.className = "inline-block w-2 h-2 rounded-full bg-slate-500";
            }
        }

        function initGamepadListener() {
            window.addEventListener("gamepadconnected", () => {
                updateGamepadStatusUI();
            });

            window.addEventListener("gamepaddisconnected", () => {
                updateGamepadStatusUI();
            });

            updateGamepadStatusUI();
            setInterval(pollGamepad, 100);
        }

        function pollGamepad() {
            updateGamepadStatusUI();
            const gamepads = navigator.getGamepads ? navigator.getGamepads() : [];
            if (!gamepads) return;

            for (let i = 0; i < gamepads.length; i++) {
                const gp = gamepads[i];
                if (gp) {
                    const buttons = gp.buttons;
                    const axes = gp.axes;

                    const now = Date.now();
                    if (!window.lastNavTime || now - window.lastNavTime > 180) {
                        let moved = false;
                        if (buttons[12]?.pressed || axes[1] < -0.6) {
                            currentFocusIndex = (currentFocusIndex - 1 + focusableElements.length) % focusableElements.length;
                            moved = true;
                        } else if (buttons[13]?.pressed || axes[1] > 0.6) {
                            currentFocusIndex = (currentFocusIndex + 1) % focusableElements.length;
                            moved = true;
                        } else if (buttons[14]?.pressed || axes[0] < -0.6) {
                            currentFocusIndex = Math.max(0, currentFocusIndex - 1);
                            moved = true;
                        } else if (buttons[15]?.pressed || axes[0] > 0.6) {
                            currentFocusIndex = Math.min(focusableElements.length - 1, currentFocusIndex + 1);
                            moved = true;
                        }

                        if (moved) {
                            window.lastNavTime = now;
                            updateFocusStyle();
                        }
                    }

                    if (buttons[4]?.pressed && !lastButtonStates[4]) {
                        switchTab('store');
                    }
                    if (buttons[5]?.pressed && !lastButtonStates[5]) {
                        switchTab('all');
                    }

                    if (buttons[0]?.pressed && !lastButtonStates[0]) {
                        if (focusableElements[currentFocusIndex]) {
                            focusableElements[currentFocusIndex].click();
                            setTimeout(refreshFocusables, 150);
                        }
                    }

                    if (buttons[1]?.pressed && !lastButtonStates[1]) {
                        const landingVisible = !document.getElementById('landingScreen').classList.contains('hidden');
                        const detailModal = document.getElementById('detailModal');
                        const queueModal = document.getElementById('queueModal');
                        const adminModal = document.getElementById('adminModal');
                        const editorModal = document.getElementById('gameEditorModal');

                        if (!landingVisible) {
                            if (!editorModal.classList.contains('hidden')) {
                                closeGameEditor();
                            } else if (!adminModal.classList.contains('hidden')) {
                                closeAdminModal();
                            } else if (!detailModal.classList.contains('hidden')) {
                                closeDetailModal();
                            } else if (!queueModal.classList.contains('hidden')) {
                                toggleQueueModal();
                            } else {
                                switchTab('store');
                            }
                        }
                    }

                    lastButtonStates = {
                        0: buttons[0]?.pressed,
                        1: buttons[1]?.pressed,
                        4: buttons[4]?.pressed,
                        5: buttons[5]?.pressed
                    };
                }
            }
        }

        function refreshFocusables() {
            const landingVisible = !document.getElementById('landingScreen').classList.contains('hidden');
            let container = document.body;
            
            if (landingVisible) {
                container = document.getElementById('landingScreen');
            } else {
                const visibleModals = document.querySelectorAll('.fixed:not(.hidden)');
                if (visibleModals.length > 0) {
                    container = visibleModals[visibleModals.length - 1];
                }
            }
            
            focusableElements = Array.from(container.querySelectorAll('.nav-focusable, button:not([disabled]), input:not([disabled]), select:not([disabled]), textarea:not([disabled]), [tabindex]:not([tabindex="-1"])'))
                .filter(el => {
                    let rect = el.getBoundingClientRect();
                    return rect.width > 0 && rect.height > 0;
                });

            if (currentFocusIndex >= focusableElements.length) {
                currentFocusIndex = 0;
            }
            updateFocusStyle();
        }

        function updateFocusStyle() {
            document.querySelectorAll('.ds4-focus').forEach(el => el.classList.remove('ds4-focus'));
            if (focusableElements.length > 0 && focusableElements[currentFocusIndex]) {
                const el = focusableElements[currentFocusIndex];
                el.classList.add('ds4-focus');
                el.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
            }
        }
    </script>
</body>
</html>
