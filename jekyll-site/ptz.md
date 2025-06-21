---
layout: default
title: Prêt à Taux Zéro (PTZ)
permalink: /ptz/
---

<h1 class="mb-4">Le Prêt à Taux Zéro (PTZ)</h1>
<p>Le PTZ est un dispositif aidé par l'État pour aider les primo-accédants à financer leur résidence principale.</p>

<h2>Qui peut en bénéficier ?</h2>
<ul>
  <li>Ne pas avoir été propriétaire dans les 2 dernières années</li>
  <li>Acheter une résidence principale (pas une location ou résidence secondaire)</li>
  <li>Respecter un plafond de ressources dépendant de la zone et du foyer</li>
</ul>

<h2>Simulateur PTZ (simplifié)</h2>
<form onsubmit="event.preventDefault(); calculerPTZ();">
  <div class="row g-3 mb-3">
    <div class="col-md-4">
      <label class="form-label">Zone</label>
      <select class="form-select" id="zone">
        <option value="A">Zone A</option>
        <option value="B1">Zone B1</option>
        <option value="B2">Zone B2</option>
        <option value="C">Zone C</option>
      </select>
    </div>
    <div class="col-md-4">
      <label class="form-label">Nombre de personnes dans le foyer</label>
      <input type="number" class="form-control" id="foyer" required>
    </div>
    <div class="col-md-4">
      <label class="form-label">Revenu fiscal de référence (€)</label>
      <input type="number" class="form-control" id="revenu" required>
    </div>
  </div>
  <button type="submit" class="btn btn-primary">Vérifier l'éligibilité</button>
</form>

<div class="mt-4" id="resultat-ptz"></div>

<script>
  const plafonds = {
    A: [37000, 51800, 62200, 72600],
    B1: [30000, 42000, 51000, 60000],
    B2: [27000, 37800, 45900, 54000],
    C: [24000, 33600, 40800, 48000]
  };

  function calculerPTZ() {
    const zone = document.getElementById("zone").value;
    const foyer = parseInt(document.getElementById("foyer").value);
    const revenu = parseInt(document.getElementById("revenu").value);
    const plafond = plafonds[zone][Math.min(foyer - 1, 3)];

    const res = document.getElementById("resultat-ptz");
    if (revenu <= plafond) {
      res.innerHTML = `<div class='alert alert-success'>✅ Vous êtes éligible au PTZ dans la zone ${zone} (plafond: ${plafond.toLocaleString()} €).</div>`;
    } else {
      res.innerHTML = `<div class='alert alert-danger'>❌ Vous n'êtes pas éligible au PTZ dans la zone ${zone} (plafond: ${plafond.toLocaleString()} €).</div>`;
    }
  }
</script>
