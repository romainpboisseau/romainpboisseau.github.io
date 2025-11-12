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
  async function loadINatObservations() {
    const gallery = document.getElementById('inat-gallery');
    let allResults = [];

    for (let page = 1; page <= 3; page++) { // get 3 pages (3 × 30 = 90)
      const response = await fetch(`https://api.inaturalist.org/v1/observations?user_id=romainpboisseau&order=desc&order_by=votes&per_page=30&page=${page}`);
      const data = await response.json();
      allResults = allResults.concat(data.results);
    }

    allResults.forEach(obs => {
      if (obs.photos && obs.photos.length > 0) {
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
        card.onclick = () => window.open(obsLink, '_blank');
        gallery.appendChild(card);
      }
    });
  }

  loadINatObservations();
</script>
