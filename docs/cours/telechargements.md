# Téléchargements

Support de cours complet et diapositives complètes, en PDF, pour chaque matière — réservés aux étudiants inscrits. **Chaque matière a son propre code d'accès**, communiqué par l'enseignant en cours.

<div class="gate-note">
Cette protection repose sur un code d'accès par matière, vérifié dans votre navigateur. Ce n'est pas un vrai contrôle d'identité côté serveur (le site est statique, sans base de données) : elle suffit à éviter qu'un lien traîne en accès libre, mais un utilisateur déterminé pourrait la contourner. Ne partagez pas les codes en dehors du cours.
</div>

<div class="module-picker" markdown>

<label for="module-select" class="picker-label">1. Choisissez votre matière</label>
<select id="module-select">
  <option value="" selected disabled>— Sélectionner une matière —</option>
</select>

<div id="module-panel" class="module-panel" style="display:none;"></div>

</div>

<script>
(function() {
  var MODULES = [
    {
      slug: "01-algorithmique-programmation-c",
      title: "1. Algorithmique et Programmation en C",
      hash: "d05364c46571ff941e0cbbadb19c8b10855216c2e5957036d073ce0e05f7aaf4",
      files: [
        ["Support de cours complet", "01-algorithmique-programmation-c-cours-complet.pdf", "517 Ko"],
        ["Diapositives complètes", "01-algorithmique-programmation-c-diapositives.pdf", "280 Ko"]
      ]
    },
    {
      slug: "02-audit-organisation-technique",
      title: "2. Audit organisation et technique",
      hash: "fca78b23d79fb6f9049239d00123171d9e0d298634c6636959fa76511e929eb9",
      files: [
        ["Support de cours complet", "02-audit-organisation-technique-cours-complet.pdf", "443 Ko"],
        ["Diapositives complètes", "02-audit-organisation-technique-diapositives.pdf", "203 Ko"]
      ]
    },
    {
      slug: "03-cryptanalyse",
      title: "3. Cryptanalyse",
      hash: "fffb6c1dcdd8ee9c9b570c02b837946d9d3994075af6ffa234b3f44e8b87312f",
      files: [
        ["Support de cours complet", "03-cryptanalyse-cours-complet.pdf", "355 Ko"],
        ["Diapositives complètes", "03-cryptanalyse-diapositives.pdf", "205 Ko"]
      ]
    },
    {
      slug: "04-gouvernance-gestion-risques-si",
      title: "4. Gouvernance et gestion des risques SI",
      hash: "60263d76c4f953a94f45f2ceff2854f3706fe42c77c87eaeb53524786ef0b79a",
      files: [
        ["Support de cours complet", "04-gouvernance-gestion-risques-si-cours-complet.pdf", "286 Ko"],
        ["Diapositives complètes", "04-gouvernance-gestion-risques-si-diapositives.pdf", "137 Ko"]
      ]
    },
    {
      slug: "05-ia-appliquee-securite-informatique",
      title: "5. IA appliquée à la sécurité informatique",
      hash: "911a25f06f12e4e350b3c935635d8d3eff1e0bd16d395b472345738dd5654edd",
      files: [
        ["Support de cours complet", "05-ia-appliquee-securite-informatique-cours-complet.pdf", "410 Ko"],
        ["Diapositives complètes", "05-ia-appliquee-securite-informatique-diapositives.pdf", "195 Ko"]
      ]
    },
    {
      slug: "06-cryptographie",
      title: "6. Cryptographie",
      hash: "1baddba504d8a08dbb0e4b9c1bf5d509e0e315d487296627242f207c4fc8f32f",
      files: [
        ["Support de cours complet", "06-cryptographie-cours-complet.pdf", "417 Ko"],
        ["Diapositives complètes", "06-cryptographie-diapositives.pdf", "227 Ko"]
      ]
    },
    {
      slug: "07-technologie-blockchain",
      title: "7. Technologie Blockchain",
      hash: "26fdd16166ac510a1813c8a112b93c493c42b2a0ae3dc059c23da604d7e27b1a",
      files: [
        ["Support de cours complet", "07-technologie-blockchain-cours-complet.pdf", "402 Ko"],
        ["Diapositives complètes", "07-technologie-blockchain-diapositives.pdf", "209 Ko"]
      ]
    },
    {
      slug: "08-security-by-design",
      title: "8. Security by design",
      hash: "d2b05b96f4b14fb34affec417180c9e1db17550acc1d522af5dc8c099e5cb831",
      files: [
        ["Support de cours complet", "08-security-by-design-cours-complet.pdf", "314 Ko"],
        ["Diapositives complètes", "08-security-by-design-diapositives.pdf", "192 Ko"]
      ]
    },
    {
      slug: "09-securite-des-applications",
      title: "9. Sécurité des applications",
      hash: "4640c1f7d87fc01f065b502194b5330be0fd59f97b1ac9a549f8d6b5be581eb8",
      files: [
        ["Support de cours complet", "09-securite-des-applications-cours-complet.pdf", "407 Ko"],
        ["Diapositives complètes", "09-securite-des-applications-diapositives.pdf", "238 Ko"]
      ]
    },
    {
      slug: "10-service-web",
      title: "10. Service web",
      hash: "5b75a08b72dc9ceb961f202695c02ba04370c897eddeddb5fdf4854c3c0e1998",
      files: [
        ["Support de cours complet", "10-service-web-cours-complet.pdf", "356 Ko"],
        ["Diapositives complètes", "10-service-web-diapositives.pdf", "327 Ko"]
      ]
    },
    {
      slug: "11-analyse-malwares-securisation-donnees",
      title: "11. Analyse de malwares et sécurisation des données",
      hash: "c194b621f8e14ca0b2bcd0a23a23bd34269b65955195214f4a270fcf199c16d9",
      files: [
        ["Support de cours complet", "11-analyse-malwares-securisation-donnees-cours-complet.pdf", "297 Ko"],
        ["Diapositives complètes", "11-analyse-malwares-securisation-donnees-diapositives.pdf", "284 Ko"]
      ]
    }
  ];

  function sha256(message) {
    var enc = new TextEncoder().encode(message);
    return crypto.subtle.digest('SHA-256', enc).then(function(buf) {
      return Array.prototype.map.call(new Uint8Array(buf), function(b) {
        return b.toString(16).padStart(2, '0');
      }).join('');
    });
  }

  var select = document.getElementById('module-select');
  var panel = document.getElementById('module-panel');

  MODULES.forEach(function(mod) {
    var opt = document.createElement('option');
    opt.value = mod.slug;
    opt.textContent = mod.title;
    select.appendChild(opt);
  });

  function isUnlocked(slug) {
    try { return sessionStorage.getItem('msi-unlocked-' + slug) === '1'; } catch (e) { return false; }
  }

  function renderPanel(mod) {
    panel.innerHTML = '';
    panel.style.display = 'block';

    var h3 = document.createElement('h3');
    h3.textContent = mod.title;
    panel.appendChild(h3);

    if (isUnlocked(mod.slug)) {
      renderLinks(mod);
      return;
    }

    var step2 = document.createElement('p');
    step2.className = 'picker-label';
    step2.textContent = '2. Saisissez votre email et le code d’accès de cette matière';
    panel.appendChild(step2);

    var form = document.createElement('form');
    form.className = 'gate-form-inline';

    var emailInput = document.createElement('input');
    emailInput.type = 'email';
    emailInput.required = true;
    emailInput.placeholder = 'Email';
    emailInput.className = 'gate-input-inline';

    var codeInput = document.createElement('input');
    codeInput.type = 'text';
    codeInput.required = true;
    codeInput.placeholder = "Code d'accès du module";
    codeInput.className = 'gate-input-inline';
    codeInput.autocomplete = 'off';

    var submitBtn = document.createElement('button');
    submitBtn.type = 'submit';
    submitBtn.className = 'md-button md-button--primary';
    submitBtn.textContent = 'Déverrouiller';

    var errorEl = document.createElement('p');
    errorEl.className = 'gate-error';

    form.appendChild(emailInput);
    form.appendChild(codeInput);
    form.appendChild(submitBtn);
    form.appendChild(errorEl);
    panel.appendChild(form);

    [emailInput, codeInput].forEach(function(el) {
      el.addEventListener('input', function() { errorEl.textContent = ''; });
    });

    form.addEventListener('submit', function(e) {
      e.preventDefault();
      var email = emailInput.value.trim();
      var code = codeInput.value.trim();
      var emailOk = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
      if (!emailOk) {
        errorEl.textContent = "Merci de saisir une adresse email valide.";
        return;
      }
      sha256(code).then(function(hash) {
        if (hash === mod.hash) {
          try { sessionStorage.setItem('msi-unlocked-' + mod.slug, '1'); } catch (e) {}
          renderPanel(mod);
        } else {
          errorEl.textContent = "Code d'accès incorrect pour cette matière.";
        }
      });
    });

    emailInput.focus();
  }

  function renderLinks(mod) {
    var linksEl = document.createElement('div');
    linksEl.className = 'download-links';
    mod.files.forEach(function(f) {
      var a = document.createElement('a');
      a.href = '../downloads/' + f[1];
      a.setAttribute('download', '');
      var tag = document.createElement('span');
      tag.className = 'pdf-tag';
      tag.textContent = 'PDF';
      a.appendChild(tag);
      a.appendChild(document.createTextNode(' ' + f[0] + ' '));
      var size = document.createElement('span');
      size.className = 'dl-size';
      size.textContent = '(' + f[2] + ')';
      a.appendChild(size);
      linksEl.appendChild(a);
    });
    panel.appendChild(linksEl);
  }

  select.addEventListener('change', function() {
    var mod = MODULES.filter(function(m) { return m.slug === select.value; })[0];
    if (mod) renderPanel(mod);
  });
})();
</script>
