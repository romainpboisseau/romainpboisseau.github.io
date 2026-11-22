---
layout: default
title: "Photography"
permalink: /photos/
---

<div style="text-align:center; margin-bottom:2rem;">
  <img src="{{ '/assets/images/photography.jpg' | relative_url }}" 
       alt="Romain Boisseau photographing wildlife"
       style="width:100%; max-height:450px; object-fit:cover; border-radius:12px; box-shadow:0 4px 20px rgba(0,0,0,0.15);">
</div>

<p class="fade-in-intro" style="max-width: 800px; margin: 1rem auto; text-align: center; font-size: 1.1em; line-height: 1.6;">
Outside of research, I enjoy observing and photographing wildlife.
Here are some of my favorite images which reflect both my appreciation for nature and my curiosity about its diversity.
You can find more on
<a href="https://www.inaturalist.org/people/romainpboisseau" target="_blank" style="color:#f2a775;text-decoration:none;font-weight:600;">iNaturalist</a>.
</p>

<style>
  /* Fade-in animation */
  .fade-in-intro {
    opacity: 0;
    animation: fadeIn 1.5s ease-out forwards;
  }

  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }

  p a:hover {
    color: #d88548; /* darker orange on hover */
    text-decoration: underline;
  }
</style>


<style>
  /* Justified rows: every photo keeps its own shape (no cropping) and
     each row is stretched to the full width, so photos in a row share
     the same height. --row-height is the target height of a row. */
  .inat-gallery {
    --row-height: 260px;
    display: flex;
    flex-wrap: wrap;
    gap: 12px;
    margin-top: 3rem;
  }

  /* Stops the last, incomplete row from being stretched while the
     script measures the rows. Removed once the gallery is trimmed so the
     last row ends exactly at the right edge. */
  .inat-gallery::after {
    content: '';
    flex-grow: 999999;
  }
  .inat-gallery.trimmed::after {
    display: none;
  }
  .inat-card.hidden {
    display: none;
  }

  @media (max-width: 600px) {
    .inat-gallery {
      --row-height: 150px;
      gap: 8px;
    }
  }

  .inat-card {
    position: relative;
    overflow: hidden;
    border-radius: 12px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.15);
    cursor: pointer;
    transition: transform 0.2s ease;
  }

  .inat-card:hover {
    transform: scale(1.03);
  }

  .inat-card {
    flex-grow: var(--ratio);
    width: calc(var(--ratio) * var(--row-height));
  }

  /* Reserves the photo's exact shape before it loads, so nothing jumps. */
  .inat-card .inat-spacer {
    display: block;
    padding-bottom: calc(100% / var(--ratio));
  }

  .inat-card img {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    object-position: 50% 50%; /* if a photo is trimmed, trim top and bottom equally */
    display: block;
  }

  .inat-meta {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    background: rgba(0,0,0,0.6);
    color: #fff;
    padding: 10px;
    font-size: 0.9em;
    opacity: 0;
    transition: opacity 0.3s ease;
  }

  .inat-card:hover .inat-meta {
    opacity: 1;
  }

  .inat-meta strong {
    display: block;
    font-size: 1em;
  }

  .inat-credit {
    margin-top: 2rem;
    font-size: 0.9em;
    text-align: center;
    color: #777;
  }
</style>

<div id="inat-gallery" class="inat-gallery"></div>

<p class="inat-credit">
  Photos by <a href="https://www.inaturalist.org/people/romainpboisseau" target="_blank">romainpboisseau</a> on iNaturalist
</p>

<script>
  function shuffle(array) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
  }

  async function loadINatObservations() {
    const gallery = document.getElementById('inat-gallery');
    gallery.innerHTML = '';

    const INAT_USER = 'romainpboisseau';
    let allResults = [];

    // Sorted by number of faves (most first). Stop once we reach
    // observations with no faves, since none of those can be ones I faved.
    for (let page = 1; page <= 30; page++) {
      const response = await fetch(
        `https://api.inaturalist.org/v1/observations?user_id=${INAT_USER}&order=desc&order_by=votes&per_page=100&page=${page}`
      );
      const data = await response.json();
      const results = data.results || [];
      allResults = allResults.concat(results);
      if (results.length < 100 || results.some(obs => (obs.faves_count || 0) === 0)) break;
    }

    // Remove duplicate observations (by obs.id)
    const uniqueMap = new Map();
    allResults.forEach(obs => uniqueMap.set(obs.id, obs));
    allResults = Array.from(uniqueMap.values());

    // Exclude specific observations
    const excludeIds = [339221532, 299172558, 337810638, 300185375, 294141089,300880256];
    allResults = allResults.filter(obs => !excludeIds.includes(obs.id));

    // Keep only observations with photos that I faved myself
    const myFaves = allResults.filter(
      obs => obs.photos && obs.photos.length > 0 &&
        (obs.faves || []).some(fave => fave.user && fave.user.login === INAT_USER)
    );

    // Rank by total number of faves (mine + other people's), most first.
    // Shuffle within each rank, then keep the first 92: the most-faved
    // photos always appear, and randomness only picks among the lowest rank that fits.
    const ranks = new Map();
    myFaves.forEach(obs => {
      const count = obs.faves_count || 0;
      if (!ranks.has(count)) ranks.set(count, []);
      ranks.get(count).push(obs);
    });
    const selected = [...ranks.keys()]
      .sort((a, b) => b - a)
      .flatMap(count => {
        const group = ranks.get(count);
        shuffle(group);
        return group;
      })
      .slice(0, 130); // a few extra, so the last row can be completed

    selected.forEach(obs => {
      const photoUrl = obs.photos[0].url.replace('square', 'large');
      const obsLink = obs.uri;
      const species = obs.species_guess || "Unknown species";
      const location = obs.place_guess || "Unknown location";

      const card = document.createElement('div');
      card.className = 'inat-card';
      // Width ÷ height of the photo, from iNaturalist (1.5 if unknown).
      const dims = obs.photos[0].original_dimensions;
      const ratio = dims && dims.width && dims.height ? dims.width / dims.height : 1.5;
      card.style.setProperty('--ratio', displayRatio(ratio));

      card.innerHTML = `
        <i class="inat-spacer"></i>
        <img src="${photoUrl}" alt="${species}" loading="lazy">
        <div class="inat-meta">
          <strong>${species}</strong>
          <span>${location}</span>
        </div>
      `;

      // If iNaturalist didn't give the size, use the real one once loaded.
      if (!(dims && dims.width && dims.height)) {
        const img = card.querySelector('img');
        img.onload = () => {
          card.style.setProperty('--ratio', displayRatio(img.naturalWidth / img.naturalHeight));
          trimToFullRows();
        };
      }

      card.onclick = () => window.open(obsLink, '_blank');
      gallery.appendChild(card);
    });

    trimToFullRows();
  }

  // Vertical photos are shown no narrower than 4:5 (width:height), so
  // taller ones lose a little at the bottom and appear larger.
  const MIN_RATIO = 0.8;
  function displayRatio(ratio) {
    return Math.max(ratio, MIN_RATIO).toFixed(4);
  }

  // Show about 92 photos, ending on a complete row so the last row fills
  // the page width. Recomputed when the window is resized.
  const TARGET_COUNT = 92;
  function trimToFullRows() {
    const gallery = document.getElementById('inat-gallery');
    const cards = [...gallery.querySelectorAll('.inat-card')];
    if (!cards.length) return;

    // Lay out every candidate photo, with the last row left ragged.
    gallery.classList.remove('trimmed');
    cards.forEach(card => card.classList.remove('hidden'));

    // Group photos into rows by their vertical position.
    const rowEnds = []; // number of photos up to the end of each row
    let lastTop = null;
    cards.forEach((card, i) => {
      const top = card.offsetTop;
      if (lastTop !== null && top !== lastTop) rowEnds.push(i);
      lastTop = top;
    });
    // rowEnds only holds complete rows: the final, ragged row is never in it.

    if (rowEnds.length) {
      // Pick the complete row whose end is closest to the target.
      const keep = rowEnds.reduce((best, n) =>
        Math.abs(n - TARGET_COUNT) < Math.abs(best - TARGET_COUNT) ? n : best);
      cards.forEach((card, i) => card.classList.toggle('hidden', i >= keep));
      gallery.classList.add('trimmed');
    }
  }

  let resizeTimer;
  window.addEventListener('resize', () => {
    clearTimeout(resizeTimer);
    resizeTimer = setTimeout(trimToFullRows, 150);
  });


  loadINatObservations();
</script>
