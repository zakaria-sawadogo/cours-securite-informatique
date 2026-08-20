# Téléchargements

Support de cours complet et diapositives complètes, en PDF, pour chacune des 9 matières — réservés aux étudiants du programme.

<div class="gate-note">
Cette protection repose sur un code d'accès unique communiqué par l'enseignant en cours, vérifié dans votre navigateur. Ce n'est pas un vrai contrôle d'identité côté serveur (le site est statique, sans base de données) : elle suffit à éviter qu'un lien traîne en accès libre, mais un utilisateur déterminé pourrait la contourner. Ne partagez pas le code en dehors du cours.
</div>

<div id="gate">
  <form id="gate-form">
    <div class="gate-field">
      <label for="gate-email">Email</label>
      <input type="email" id="gate-email" required placeholder="prenom.nom@exemple.com" autocomplete="email">
    </div>
    <div class="gate-field">
      <label for="gate-code">Code d'accès</label>
      <input type="text" id="gate-code" required placeholder="Communiqué en cours" autocomplete="off">
    </div>
    <button type="submit" class="md-button md-button--primary">Déverrouiller les téléchargements</button>
    <p id="gate-error" class="gate-error"></p>
  </form>
</div>

<div id="downloads" style="display:none">

<div class="download-group">
<h3><span class="dl-num">1</span> Algorithmique et Programmation en C</h3>
<div class="download-links">
<a href="../downloads/01-algorithmique-programmation-c-cours-complet.pdf" download><span class="pdf-tag">PDF</span> Support de cours complet <span class="dl-size">(517 Ko)</span></a>
<a href="../downloads/01-algorithmique-programmation-c-diapositives.pdf" download><span class="pdf-tag">PDF</span> Diapositives complètes <span class="dl-size">(280 Ko)</span></a>
</div>
</div>

<div class="download-group">
<h3><span class="dl-num">2</span> Audit organisation et technique</h3>
<div class="download-links">
<a href="../downloads/02-audit-organisation-technique-cours-complet.pdf" download><span class="pdf-tag">PDF</span> Support de cours complet <span class="dl-size">(443 Ko)</span></a>
<a href="../downloads/02-audit-organisation-technique-diapositives.pdf" download><span class="pdf-tag">PDF</span> Diapositives complètes <span class="dl-size">(203 Ko)</span></a>
</div>
</div>

<div class="download-group">
<h3><span class="dl-num">3</span> Cryptanalyse</h3>
<div class="download-links">
<a href="../downloads/03-cryptanalyse-cours-complet.pdf" download><span class="pdf-tag">PDF</span> Support de cours complet <span class="dl-size">(355 Ko)</span></a>
<a href="../downloads/03-cryptanalyse-diapositives.pdf" download><span class="pdf-tag">PDF</span> Diapositives complètes <span class="dl-size">(205 Ko)</span></a>
</div>
</div>

<div class="download-group">
<h3><span class="dl-num">4</span> Gouvernance et gestion des risques SI</h3>
<div class="download-links">
<a href="../downloads/04-gouvernance-gestion-risques-si-cours-complet.pdf" download><span class="pdf-tag">PDF</span> Support de cours complet <span class="dl-size">(286 Ko)</span></a>
<a href="../downloads/04-gouvernance-gestion-risques-si-diapositives.pdf" download><span class="pdf-tag">PDF</span> Diapositives complètes <span class="dl-size">(137 Ko)</span></a>
</div>
</div>

<div class="download-group">
<h3><span class="dl-num">5</span> IA appliquée à la sécurité informatique</h3>
<div class="download-links">
<a href="../downloads/05-ia-appliquee-securite-informatique-cours-complet.pdf" download><span class="pdf-tag">PDF</span> Support de cours complet <span class="dl-size">(410 Ko)</span></a>
<a href="../downloads/05-ia-appliquee-securite-informatique-diapositives.pdf" download><span class="pdf-tag">PDF</span> Diapositives complètes <span class="dl-size">(195 Ko)</span></a>
</div>
</div>

<div class="download-group">
<h3><span class="dl-num">6</span> Cryptographie</h3>
<div class="download-links">
<a href="../downloads/06-cryptographie-cours-complet.pdf" download><span class="pdf-tag">PDF</span> Support de cours complet <span class="dl-size">(417 Ko)</span></a>
<a href="../downloads/06-cryptographie-diapositives.pdf" download><span class="pdf-tag">PDF</span> Diapositives complètes <span class="dl-size">(227 Ko)</span></a>
</div>
</div>

<div class="download-group">
<h3><span class="dl-num">7</span> Technologie Blockchain</h3>
<div class="download-links">
<a href="../downloads/07-technologie-blockchain-cours-complet.pdf" download><span class="pdf-tag">PDF</span> Support de cours complet <span class="dl-size">(402 Ko)</span></a>
<a href="../downloads/07-technologie-blockchain-diapositives.pdf" download><span class="pdf-tag">PDF</span> Diapositives complètes <span class="dl-size">(209 Ko)</span></a>
</div>
</div>

<div class="download-group">
<h3><span class="dl-num">8</span> Security by design</h3>
<div class="download-links">
<a href="../downloads/08-security-by-design-cours-complet.pdf" download><span class="pdf-tag">PDF</span> Support de cours complet <span class="dl-size">(314 Ko)</span></a>
<a href="../downloads/08-security-by-design-diapositives.pdf" download><span class="pdf-tag">PDF</span> Diapositives complètes <span class="dl-size">(192 Ko)</span></a>
</div>
</div>

<div class="download-group">
<h3><span class="dl-num">9</span> Sécurité des applications</h3>
<div class="download-links">
<a href="../downloads/09-securite-des-applications-cours-complet.pdf" download><span class="pdf-tag">PDF</span> Support de cours complet <span class="dl-size">(407 Ko)</span></a>
<a href="../downloads/09-securite-des-applications-diapositives.pdf" download><span class="pdf-tag">PDF</span> Diapositives complètes <span class="dl-size">(238 Ko)</span></a>
</div>
</div>

</div>

<script>
(function() {
  // Code d'accès par défaut : EPO-MSI-2026 (à changer ci-dessous, voir le README du dépôt)
  var CODE_HASH = "231669d3bd04b7e0d1c65cba2b323f08ba4c217262aef169c442675f139e5b82";

  function sha256(message) {
    var enc = new TextEncoder().encode(message);
    return crypto.subtle.digest('SHA-256', enc).then(function(buf) {
      return Array.prototype.map.call(new Uint8Array(buf), function(b) {
        return b.toString(16).padStart(2, '0');
      }).join('');
    });
  }

  var form = document.getElementById('gate-form');
  var errorEl = document.getElementById('gate-error');
  var gateEl = document.getElementById('gate');
  var downloadsEl = document.getElementById('downloads');

  function unlock() {
    gateEl.style.display = 'none';
    downloadsEl.style.display = 'block';
  }

  ['gate-email', 'gate-code'].forEach(function(id) {
    var el = document.getElementById(id);
    if (el) el.addEventListener('input', function() { errorEl.textContent = ''; });
  });

  if (form) {
    form.addEventListener('submit', function(e) {
      e.preventDefault();
      var email = document.getElementById('gate-email').value.trim();
      var code = document.getElementById('gate-code').value.trim();
      var emailOk = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
      if (!emailOk) {
        errorEl.textContent = "Merci de saisir une adresse email valide.";
        return;
      }
      if (!code) {
        errorEl.textContent = "Merci de saisir le code d'accès.";
        return;
      }
      sha256(code).then(function(hash) {
        if (hash === CODE_HASH) {
          errorEl.textContent = "";
          try { sessionStorage.setItem('msi-downloads-unlocked', '1'); } catch (e) {}
          unlock();
        } else {
          errorEl.textContent = "Code d'accès incorrect. Demandez-le à votre enseignant.";
        }
      });
    });
  }

  try {
    if (sessionStorage.getItem('msi-downloads-unlocked') === '1') {
      unlock();
    }
  } catch (e) {}
})();
</script>
