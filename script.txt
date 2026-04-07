'use strict';

const tg = window.Telegram?.WebApp ?? null;
if (tg) {
  tg.ready();
  tg.expand();
  tg.BackButton.onClick(() => {
    if (!modal.hidden) closeModal();
    else tg.BackButton.hide();
  });
}

const KODIK_TOKEN = '631980894562086381467438466'; 
const API_URL = 'https://kodikapi.com/search';

const grid = document.getElementById('mediaGrid');
const searchInput = document.getElementById('searchInput');
const searchClear = document.getElementById('searchClear');
const stateLoading = document.getElementById('stateLoading');
const stateEmpty = document.getElementById('stateEmpty');

const modal = document.getElementById('modal');
const modalClose = document.getElementById('modalClose');
const modalTitle = document.getElementById('modalTitle');
const modalOverview = document.getElementById('modalOverview');
const modalRating = document.getElementById('modalRating');
const modalYear = document.getElementById('modalYear');
const modalGenre = document.getElementById('modalGenre');
const modalPoster = document.getElementById('modalPoster');
const playerFrame = document.getElementById('player');
const playerPlaceholder = document.getElementById('playerPlaceholder');

let debounceTimer = null;

async function loadMedia(query = '') {
    if (!query) query = '2024'; 
    setState('loading');
    try {
        const response = await fetch(`${API_URL}?token=${KODIK_TOKEN}&title=${encodeURIComponent(query)}&limit=50&with_material_data=true`);
        const data = await response.json();
        if (!data.results || data.results.length === 0) {
            setState('empty');
            return;
        }
        renderGrid(data.results);
    } catch (err) {
        console.error("Ошибка:", err);
        setState('empty');
    }
}

function renderGrid(items) {
    grid.innerHTML = '';
    setState('grid');
    items.forEach((item, i) => {
        const card = document.createElement('article');
        card.className = 'card';
        const poster = item.material_data?.poster_url || 'https://placehold.co/300x450/1a1a22/7c5cfc?text=No+Poster';
        const rating = item.material_data?.kinopoisk_rating || '—';
        const year = item.year || item.material_data?.year || '—';
        card.innerHTML = `
            <div class="card-poster-wrap">
                <img class="card-poster" src="${poster}" loading="lazy" />
                <div class="card-rating">★ ${rating}</div>
            </div>
            <div class="card-body">
                <p class="card-title">${item.title}</p>
                <p class="card-sub">${year}</p>
            </div>`;
        card.onclick = () => openModal(item);
        grid.appendChild(card);
    });
}

function openModal(item) {
    modalTitle.textContent = item.title;
    modalOverview.textContent = item.material_data?.description || "Описание отсутствует.";
    modalRating.textContent = item.material_data?.kinopoisk_rating || "—";
    modalYear.textContent = item.year || item.material_data?.year || "—";
    modalGenre.textContent = item.material_data?.genres ? item.material_data.genres[0] : "Кино";
    modalPoster.src = item.material_data?.poster_url || '';
    playerFrame.src = '';
    playerFrame.hidden = true;
    playerPlaceholder.hidden = false;
    playerPlaceholder.onclick = () => {
        playerFrame.src = item.link;
        playerFrame.hidden = false;
        playerPlaceholder.hidden = true;
    };
    modal.hidden = false;
    document.body.style.overflow = 'hidden';
    if (tg) tg.BackButton.show();
}

function closeModal() {
    modal.hidden = true;
    document.body.style.overflow = '';
    playerFrame.src = '';
    if (tg) tg.BackButton.hide();
}

function setState(state) {
    stateLoading.hidden = state !== 'loading';
    stateEmpty.hidden = state !== 'empty';
    grid.hidden = state !== 'grid';
}

searchInput.addEventListener('input', (e) => {
    const q = e.target.value;
    searchClear.hidden = q.length === 0;
    clearTimeout(debounceTimer);
    debounceTimer = setTimeout(() => { if (q.length > 2) loadMedia(q); }, 600);
});

searchClear.onclick = () => {
    searchInput.value = '';
    searchClear.hidden = true;
    loadMedia();
};

modalClose.onclick = closeModal;
loadMedia();
