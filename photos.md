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
  .inat-gallery {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 15px;
    margin-top: 3rem;
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

  .inat-card img {
    width: 100%;
    height: 280px;
    object-fit: cover;
    display: block;
  }

  /* Vertical photos: crop toward the top instead of the center.
     0% = very top, 50% = center. */
  .inat-card img.portrait {
    object-position: center 20%;
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

<script src="https://cdn.jsdelivr.net/npm/smartcrop@2.0.5/smartcrop.js"></script>
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
      .slice(0, 92);

    const cropQueue = [];

    selected.forEach(obs => {
      const photoUrl = obs.photos[0].url.replace('square', 'large');
      const obsLink = obs.uri;
      const species = obs.species_guess || "Unknown species";
      const location = obs.place_guess || "Unknown location";

      const card = document.createElement('div');
      card.className = 'inat-card';
      card.innerHTML = `
        <img src="${photoUrl}" alt="${species}">
        <div class="inat-meta">
          <strong>${species}</strong>
          <span>${location}</span>
        </div>
      `;

      const img = card.querySelector('img');
      img.onload = () => {
        if (img.naturalHeight > img.naturalWidth) img.classList.add('portrait');
      };

      card.onclick = () => window.open(obsLink, '_blank');
      gallery.appendChild(card);
      // Only vertical photos get content-aware cropping. Skip photos that
      // iNaturalist already tells us are horizontal or square.
      const dims = obs.photos[0].original_dimensions;
      if (!dims || dims.height > dims.width) {
        cropQueue.push({ img, smallUrl: obs.photos[0].url.replace('square', 'small') });
      }
    });

    smartCropAll(cropQueue);
  }

  // Content-aware cropping with smartcrop.js, for vertical photos only: it looks for the most
  // interesting part of each photo (detail, skin tones, saturation) and
  // centers the crop on it. It works on a small copy of each photo, one at
  // a time, after the gallery is shown. If anything fails (library not
  // loaded, photo host blocks it), the photo keeps the default crop.
  async function smartCropAll(items) {
    if (!window.smartcrop) return;
    for (const { img, smallUrl } of items) {
      try {
        const probe = new Image();
        probe.crossOrigin = 'anonymous';
        probe.src = smallUrl;
        await probe.decode();
        if (probe.naturalHeight <= probe.naturalWidth) continue; // not vertical
        const box = img.getBoundingClientRect();
        if (!box.width || !box.height) continue;
        const { topCrop } = await smartcrop.crop(probe, { width: box.width, height: box.height });
        const freeY = probe.naturalHeight - topCrop.height;
        if (freeY > 1) {
          const posY = (topCrop.y / freeY) * 100;
          img.style.objectPosition = `50% ${posY.toFixed(1)}%`;
        }
      } catch (e) {
        // keep the default crop
      }
      await new Promise(resolve => setTimeout(resolve, 0)); // keep the page responsive
    }
  }

  loadINatObservations();
</script>
