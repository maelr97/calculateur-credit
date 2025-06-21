---
layout: default
title: Simulateur
permalink: /simulateur/
---

<div class="row">
  <div class="col-md-6">
    <form id="form-simulation">
      <div class="mb-3">
        <label for="prix" class="form-label">Prix du bien immobilier (€)</label>
        <input type="number" class="form-control" id="prix" placeholder="Ex. : 250000" required>
      </div>
      <div class="mb-3">
        <label for="apport" class="form-label">Apport personnel (€)</label>
        <input type="number" class="form-control" id="apport" placeholder="Ex. : 30000" required>
      </div>
      <div class="mb-3">
        <label for="taux" class="form-label">Taux d'intérêt annuel (%)</label>
        <input type="number" step="0.01" class="form-control" id="taux" placeholder="Ex. : 2.5" required>
      </div>
      <div class="mb-3">
        <label for="assurance" class="form-label">Taux assurance annuelle (%)</label>
        <input type="number" step="0.01" class="form-control" id="assurance" placeholder="Ex. : 0.36" required>
      </div>
      <div class="mb-3">
        <label for="duree" class="form-label">Durée du prêt (années)</label>
        <input type="number" class="form-control" id="duree" placeholder="Ex. : 20" required>
      </div>
      <button type="button" class="btn btn-primary w-100" onclick="calculer()">Calculer</button>
    </form>
  </div>

  <div class="col-md-6">
    <div class="card text-center p-4 shadow-sm">
      <h5 class="mb-3">Votre mensualité sera de</h5>
      <h1 class="display-4 text-warning" id="mensualite-resultat">-- €</h1>
      <hr>
      <p><strong>Montant de votre prêt :</strong> <span id="montant-emprunte">--</span>€</p>
      <p><strong>Votre mensualité :</strong> <span id="mensualite-credit">--</span>€/mois<br><small>dont assurance : <span id="mensualite-assurance">--</span>€</small></p>
      <p><strong>Coût total du crédit :</strong> <span id="cout-total">--</span>€<br><small>dont assurance : <span id="cout-assurance">--</span>€</small></p>
    </div>
  </div>
</div>

<div class="mt-5">
  <canvas id="amortissementChart" height="100"></canvas>
</div>

<div class="text-center mt-4">
  <button id="btn-detail" class="btn btn-outline-secondary" onclick="toggleTableau()" style="display:none;">Afficher le détail</button>
</div>
<div id="tableau-container" class="mt-3 collapsed" style="display:none;">
  <div id="tableau"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  let chartInstance;

  function calculer() {
    const prix = parseFloat(document.getElementById('prix').value);
    const apport = parseFloat(document.getElementById('apport').value);
    const tauxAnnuel = parseFloat(document.getElementById('taux').value);
    const assuranceAnnuel = parseFloat(document.getElementById('assurance').value);
    const duree = parseInt(document.getElementById('duree').value);

    if (
      isNaN(prix) || isNaN(apport) || isNaN(tauxAnnuel) ||
      isNaN(assuranceAnnuel) || isNaN(duree) ||
      prix <= 0 || apport < 0 || tauxAnnuel <= 0 || assuranceAnnuel < 0 || duree <= 0
    ) {
      alert('Veuillez remplir tous les champs correctement.');
      return;
    }

    const montantEmprunte = prix - apport;
    if (montantEmprunte <= 0) {
      alert("L'apport ne peut pas être supérieur ou égal au prix du bien.");
      return;
    }

    const tauxMensuel = tauxAnnuel / 100 / 12;
    const tauxAssuranceMensuel = assuranceAnnuel / 100 / 12;
    const nbMensualites = duree * 12;

    const mensualiteCredit = montantEmprunte * tauxMensuel / (1 - Math.pow(1 + tauxMensuel, -nbMensualites));
    const mensualiteAssurance = montantEmprunte * tauxAssuranceMensuel;
    const mensualiteTotale = mensualiteCredit + mensualiteAssurance;

    const coutCredit = mensualiteCredit * nbMensualites - montantEmprunte;
    const coutAssurance = mensualiteAssurance * nbMensualites;
    const coutTotal = coutCredit + coutAssurance;

    document.getElementById('btn-detail').style.display = 'inline-block';

    document.getElementById('montant-emprunte').textContent = montantEmprunte.toFixed(0);
    document.getElementById('mensualite-credit').textContent = mensualiteCredit.toFixed(0);
    document.getElementById('mensualite-assurance').textContent = mensualiteAssurance.toFixed(0);
    document.getElementById('mensualite-resultat').textContent = mensualiteTotale.toFixed(0) + '€';
    document.getElementById('cout-total').textContent = coutTotal.toFixed(0);
    document.getElementById('cout-assurance').textContent = coutAssurance.toFixed(0);

    let capitalRestant = montantEmprunte;
    let cumulInterets = 0;
    let cumulCapital = 0;

    const labels = [], capitalRestantData = [], interetsData = [], capitalRembourseData = [];
    let tableau = `<div class="table-responsive"><table class="table table-bordered table-sm mt-3">
      <thead class="table-light"><tr><th>Mois</th><th>Intérêts</th><th>Capital</th><th>Restant</th></tr></thead><tbody>`;

    for (let i = 1; i <= nbMensualites; i++) {
      const interets = capitalRestant * tauxMensuel;
      const capitalRembourse = mensualiteCredit - interets;
      capitalRestant -= capitalRembourse;

      cumulInterets += interets;
      cumulCapital += capitalRembourse;

      labels.push(i);
      capitalRestantData.push(Math.max(capitalRestant, 0));
      interetsData.push(cumulInterets);
      capitalRembourseData.push(cumulCapital);

      tableau += `<tr><td>${i}</td><td>${interets.toFixed(2)}</td><td>${capitalRembourse.toFixed(2)}</td><td>${Math.max(capitalRestant, 0).toFixed(2)}</td></tr>`;
    }

    tableau += `</tbody></table></div>`;
    document.getElementById('tableau').innerHTML = tableau;
    document.getElementById('tableau-container').style.display = 'none';
    document.getElementById('btn-detail').textContent = 'Afficher le détail';

    if (window.chartInstance) chartInstance.destroy();
    const ctx = document.getElementById('amortissementChart').getContext('2d');
    chartInstance = new Chart(ctx, {
      type: 'line',
      data: {
        labels: labels,
        datasets: [
          {
            label: 'Capital restant dû',
            data: capitalRestantData,
            borderColor: 'black',
            fill: false
          },
          {
            label: 'Capital remboursé (cumulé)',
            data: capitalRembourseData,
            borderColor: 'green',
            backgroundColor: 'rgba(0,128,0,0.2)',
            fill: true
          },
          {
            label: 'Intérêts payés (cumulés)',
            data: interetsData,
            borderColor: 'orange',
            backgroundColor: 'rgba(255,165,0,0.3)',
            fill: true
          }
        ]
      },
      options: {
        responsive: true,
        scales: {
          y: { beginAtZero: true, title: { display: true, text: 'Montant (€)' } },
          x: { title: { display: true, text: 'Durée (mois)' } }
        }
      }
    });
  }

  function toggleTableau() {
    const container = document.getElementById('tableau-container');
    if (container.style.display === 'none') {
      container.style.display = 'block';
      document.getElementById('btn-detail').textContent = 'Masquer le détail';
    } else {
      container.style.display = 'none';
      document.getElementById('btn-detail').textContent = 'Afficher le détail';
    }
  }
</script>
