---
search:
  exclude: true
---

# Générateur de codes d'accès — admin

<div id="admin-gate">

<div class="module-picker" markdown>

<label for="admin-password" class="picker-label">Mot de passe administrateur</label>
<div class="gate-form-inline">
<input type="password" id="admin-password" class="gate-input-inline" placeholder="Mot de passe" autocomplete="off">
<button type="button" id="admin-unlock" class="md-button md-button--primary">Entrer</button>
</div>
<p class="gate-error" id="admin-error"></p>

</div>

</div>

<div id="admin-content" style="display:none;" markdown>

Choisissez une matière, saisissez un nouveau code, récupérez la ligne à coller dans `docs/cours/telechargements.md` (dans le tableau `MODULES`, à l'intérieur du bloc de script en bas du fichier).

<div class="module-picker" markdown>

<label for="gen-matiere" class="picker-label">Matière</label>
<select id="gen-matiere"></select>

<label for="gen-code" class="picker-label">Nouveau code d'accès</label>
<div class="gate-form-inline">
<input type="text" id="gen-code" class="gate-input-inline" placeholder="ex. EPO-01-PROMO2027" autocomplete="off">
<button type="button" id="gen-generate" class="md-button md-button--primary">Générer</button>
</div>

<div id="gen-result" style="display:none; margin-top:1rem;">
<p class="picker-label">Code à communiquer aux étudiants</p>
<div class="admin-code-out" id="gen-out-code"></div>
<p class="picker-label" style="margin-top:0.8rem;">Ligne à coller dans telechargements.md pour cette matière</p>
<div class="admin-snippet" id="gen-out-snippet"></div>
</div>

</div>

## Codes actuellement en ligne

| Matière | Code par défaut |
|---|---|
| 1. Algorithmique et Programmation en C | EPO-01-2026 |
| 2. Audit organisation et technique | EPO-02-2026 |
| 3. Cryptanalyse | EPO-03-2026 |
| 4. Gouvernance et gestion des risques SI | EPO-04-2026 |
| 5. IA appliquée à la sécurité informatique | EPO-05-2026 |
| 6. Cryptographie | EPO-06-2026 |
| 7. Technologie Blockchain | EPO-07-2026 |
| 8. Security by design | EPO-08-2026 |
| 9. Sécurité des applications | EPO-09-2026 |
| 10. Service web | EPO-10-2026 |
| 11. Analyse de malwares et sécurisation des données | EPO-11-2026 |

*(Cette liste reflète les codes par défaut mis en place initialement — elle ne se met pas à jour automatiquement quand vous en changez un ici. Notez vos changements de votre côté.)*

</div>

<script>
(function() {
  var ADMIN_HASH = "5ad5680083f16b062708bec5d753d33f88d8994d75e92d6df113b553e20e8cf0";

  var MATIERES = [
    ["01-algorithmique-programmation-c", "1. Algorithmique et Programmation en C"],
    ["02-audit-organisation-technique", "2. Audit organisation et technique"],
    ["03-cryptanalyse", "3. Cryptanalyse"],
    ["04-gouvernance-gestion-risques-si", "4. Gouvernance et gestion des risques SI"],
    ["05-ia-appliquee-securite-informatique", "5. IA appliquée à la sécurité informatique"],
    ["06-cryptographie", "6. Cryptographie"],
    ["07-technologie-blockchain", "7. Technologie Blockchain"],
    ["08-security-by-design", "8. Security by design"],
    ["09-securite-des-applications", "9. Sécurité des applications"],
    ["10-service-web", "10. Service web"],
    ["11-analyse-malwares-securisation-donnees", "11. Analyse de malwares et sécurisation des données"]
  ];

  function sha256(message) {
    var enc = new TextEncoder().encode(message);
    return crypto.subtle.digest('SHA-256', enc).then(function(buf) {
      return Array.prototype.map.call(new Uint8Array(buf), function(b) {
        return b.toString(16).padStart(2, '0');
      }).join('');
    });
  }

  var select = document.getElementById('gen-matiere');
  MATIERES.forEach(function(m) {
    var opt = document.createElement('option');
    opt.value = m[0];
    opt.textContent = m[1];
    select.appendChild(opt);
  });

  document.getElementById('admin-unlock').addEventListener('click', function() {
    var pw = document.getElementById('admin-password').value;
    var errorEl = document.getElementById('admin-error');
    if (!pw) return;
    sha256(pw).then(function(hash) {
      if (hash === ADMIN_HASH) {
        document.getElementById('admin-gate').style.display = 'none';
        document.getElementById('admin-content').style.display = 'block';
        try { sessionStorage.setItem('msi-admin-unlocked', '1'); } catch (e) {}
      } else {
        errorEl.textContent = "Mot de passe incorrect.";
      }
    });
  });

  document.getElementById('admin-password').addEventListener('keydown', function(e) {
    if (e.key === 'Enter') { document.getElementById('admin-unlock').click(); }
  });

  document.getElementById('gen-generate').addEventListener('click', function() {
    var slug = select.value;
    var code = document.getElementById('gen-code').value.trim();
    if (!code) { alert("Saisissez d'abord un code."); return; }
    sha256(code).then(function(hash) {
      document.getElementById('gen-out-code').textContent = code;
      document.getElementById('gen-out-snippet').textContent =
        'slug: "' + slug + '"\nhash: "' + hash + '"';
      document.getElementById('gen-result').style.display = 'block';
    });
  });

  try {
    if (sessionStorage.getItem('msi-admin-unlocked') === '1') {
      document.getElementById('admin-gate').style.display = 'none';
      document.getElementById('admin-content').style.display = 'block';
    }
  } catch (e) {}
})();
</script>
