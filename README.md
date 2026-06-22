<html lang="fr">

<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta http-equiv="Content-Security-Policy" content="default-src 'none';
		script-src 'self' 'unsafe-inline';
		style-src 'unsafe-inline';
connect-src 'self'
  https://geo.api.gouv.fr
  https://api-lannuaire.service-public.fr
  https://raw.githubusercontent.com
  https://static.data.gouv.fr
  https://www.data.gouv.fr;
frame-ancestors 'none';">

	<meta http-equiv="X-Content-Type-Options" content="nosniff">
	<meta name="referrer" content="strict-origin">
	<meta http-equiv="Strict-Transport-Security" content="max-age=63072000; includeSubDomains; preload">


	<title>Recherche d'une commune</title>

	<style id="af">html { display: none !important; }</style>
<script>
  if (self === top) {
    var s = document.getElementById('af');
    if (s) s.remove();            // pas framé → on affiche
  } else {
    top.location = self.location; // framé → tentative de sortie
  }
</script>


	<style>
	body {
		font-family: 'tahoma', 'Helvetica', 'Arial', sans-serif;
	}
	
	.combobox {
		position: relative;
		display: inline-block;
		width: 200px;
	}
	
	.combobox input {
		width: 100%;
		padding: 5px;
		border: 1px solid #ccc;
	}

	.combobox .dropdown-menu li[aria-selected="true"] {
    background-color: #ddd;
}
	.combobox .dropdown-menu {
		display: none;
		position: absolute;
		top: 100%;
		left: 0;
		width: 100%;
		margin: 0;
		padding: 0;
		list-style: none;
		background-color: #f1f1f1;
		z-index: 5;
	}
	
	.combobox .dropdown-menu li {
		padding: 5px;
		cursor: pointer;
	}
	
	.combobox .dropdown-menu li:hover {
		background-color: #ddd;
	}
	
	.combobox {
		margin-right: 20px;
	}
	
	.error-message {
		color: red;
	}
	
	table {
		display: none;
		border-collapse: collapse;
	}
	
	table,
	th,
	td {
		border: 1px solid black;
	}
	
	th,
	td {
		padding-right: 5px;
		padding-left: 5px;
		text-align: left;
	}

	#chargement {
    display: none;
    margin-top: 10px;
    color: #555;
}

.spinner {
    display: inline-block;
    width: 14px;
    height: 14px;
    border: 2px solid #ccc;
    border-top-color: #555;
    border-radius: 50%;
    animation: spin 0.7s linear infinite;
    vertical-align: middle;
    margin-right: 6px;
}

@keyframes spin {
    to { transform: rotate(360deg); }
}
	</style>
</head>

<body>
	<h1>Recherche d'une commune</h1>
<div class="combobox">
    <label for="communeInput">Commune</label>
    <input 
        type="text" 
        id="communeInput" 
        name="commune" 
        autocomplete="off"
        role="combobox"
        aria-expanded="false"
        aria-controls="commune-list"
        aria-autocomplete="list"
        aria-haspopup="listbox"
        aria-activedescendant="">
    <ul 
        id="commune-list" 
        class="dropdown-menu" 
        role="listbox" 
        aria-label="Communes suggérées">
    </ul>
</div>
	<button id="rechercherBtn">Rechercher</button>
<div id="chargement" aria-live="polite" aria-busy="false">
    <span class="spinner" aria-hidden="true"></span>
    Chargement en cours…
</div>
	<div id="resultatCommune"></div>
	<div id="infos"></div>
	<table>
		<tr>
			<th colspan="2" style="text-align: left;"><b>– Population –</b></th>
		</tr>
		<tr>
			<td><b>Population de la commune <sup>(1)</sup></b></td>
			<td id="populationInfo"></td>
		</tr>
		<tr>
			<td><b>Population de l'unité urbaine <sup>(2)</sup></b></td>
			<td id="popUrbaineInfo"></td>
		</tr>
	</table>
	<br>
	<br>
	<table>
		<tr>
			<th colspan="2" style="text-align: left;"><b>– Mairie –</b></th>
		</tr>
		<tr>
			<td><b>Nom du maire <sup>(3)</sup></b></td>
			<td id="nomdumaire"></td>
		</tr>
		<tr>
			<td><b>Adresse de la mairie <sup>(4)</sup></b></td>
			<td id="adressemairie"></td>
		</tr>
		<tr>
			<td><b>Adresse courriel de la mairie <sup>(4)</sup></b></td>
			<td id="courrielmairie"></td>
		</tr>
		<tr>
			<td><b>Site internet de la mairie <sup>(4)</sup></b></td>
			<td id="sitemairie"></td>
		</tr>
	</table>
	<br>
	<br>
	<table>
		<tr>
			<th colspan="2" style="text-align: left;"><b>– EPCI –</b></th>
		</tr>
		<tr>
			<td><b>EPCI <sup>(1)</sup></b></td>
			<td id="epciInfo"></td>
		</tr>
		<tr>
			<td><b>Nom du président <sup>(3)</sup></b></td>
			<td id="nomdupresident"></td>
		</tr>
		<tr>
			<td><b>Adresse de l'EPCI <sup>(4)</sup></b></td>
			<td id="adresseEpci"></td>
		</tr>
		<tr>
			<td><b>Adresse courriel de l'EPCI <sup>(4)</sup></b></td>
			<td id="courrielEpci"></td>
		</tr>
		<tr>
			<td><b>Site internet de l'EPCI <sup>(4)</sup></b></td>
			<td id="siteEpci"></td>
		</tr>
		<tr>
			<td><b>Compétence PLU <sup>(5)</sup></b></td>
			<td id="competencePLU"></td>
		</tr>
		<tr>
			<td><b>Compétence RLP <sup>(5)</sup></b></td>
			<td id="competenceRLP"></td>
		</tr>
	</table>
	<br>
	<script>
	document.addEventListener('DOMContentLoaded', function() {
const communeInput = document.getElementById("communeInput");
const communeList = document.getElementById("commune-list");
const rechercherBtn = document.getElementById("rechercherBtn");
const infosElement = document.getElementById("infos");
		let selectedCodeCommune;
		let activeIndex = -1;
		let communeController = null;
 const csvCache = {};
 const SIREN_MGP = "200054781";
 let latestFetchId = 0;

function setTextIfCurrent(fetchId, elementId, text) {
    if (fetchId !== latestFetchId) return;
    const element = document.getElementById(elementId);
    if (!element) {
        console.warn(`Élément introuvable : ${elementId}`);
        return;
    }
    element.textContent = typeof text === 'string'
        ? normalizeText(text)
        : 'Données non disponibles';
}

/**
 * Normalise un code INSEE ou SIREN pour la comparaison.
 * Nécessaire car les sources sont incohérentes : l'API geo.gouv.fr
 * retourne "01001" tandis que les CSV du RNE peuvent retourner "1001".
 * Les codes corses (2A, 2B) ne commençant pas par zéro, ils ne sont pas affectés.
 */
function normalizeCode(c) {
    if (c == null) return '';
    return String(c).trim().replace(/^0+/, '');
}

function selectionnerCommune(commune) {
    selectedCodeCommune = commune.code;
    communeInput.value = commune.nom;
    hideCommuneList();
    infosElement.textContent = '';

    const ids = ['populationInfo', 'popUrbaineInfo', 'epciInfo', 'nomdumaire',
                 'adressemairie', 'courrielmairie', 'sitemairie', 'nomdupresident',
                 'adresseEpci', 'courrielEpci', 'siteEpci', 'competencePLU', 'competenceRLP'];
    ids.forEach(id => document.getElementById(id).textContent = '');

    const resultatCommune = document.getElementById('resultatCommune');
    const h2Element = document.createElement('h2');
    h2Element.textContent = normalizeText(
        `– ${commune.nom} (${commune.codeDepartement}) – code INSEE ${commune.code}`);
    resultatCommune.textContent = '';
    resultatCommune.appendChild(h2Element);
}

function showLoading() {
    const el = document.getElementById('chargement');
    el.style.display = 'block';
    el.setAttribute('aria-busy', 'true');
    rechercherBtn.disabled = true;
}

function hideLoading() {
    const el = document.getElementById('chargement');
    el.style.display = 'none';
    el.setAttribute('aria-busy', 'false');
    rechercherBtn.disabled = false;
}


function safeJsonParse(value, fallback = null) {
    try {
        return JSON.parse(value);
    } catch (e) {
        console.warn("JSON invalide :", value, e);
        return fallback;
    }
}


async function fetchWithTimeout(url, options = {}, timeout = 12000) {
    const controller = new AbortController();
    const onExternalAbort = () => controller.abort();

    // Abandon après `timeout` ms, avec un nom d'erreur distinct
    // pour rester compatible avec le catch de fetchCommunes.
    const timeoutId = setTimeout(
        () => controller.abort(new DOMException('Délai dépassé', 'TimeoutError')),
        timeout
    );

    // Propage un éventuel signal externe (ex. communeController)
    if (options.signal) {
        if (options.signal.aborted) {
            controller.abort();
        } else {
            options.signal.addEventListener('abort', onExternalAbort, { once: true });
        }
    }

    try {
        return await fetch(url, { ...options, signal: controller.signal });
    } finally {
        clearTimeout(timeoutId);
        if (options.signal) options.signal.removeEventListener('abort', onExternalAbort);
    }
}

async function fetchCsvData(url) {
    if (csvCache[url]) {
        return csvCache[url];
    }
    try {
		const response = await fetchWithTimeout(url);

        if (!response.ok) {
            throw new Error(`Erreur réseau : ${response.status}`);
        }

        const text = await response.text();

        const data = parseCsv(text);


        const result = data;
        csvCache[url] = result;
        return result;

    } catch (error) {

        console.error(error);

        return null;
    }
}



function parseCsv(text, separator = ';') {
    const rows = [];
    let row = [];
    let current = '';
    let insideQuotes = false;
    let fieldHadQuotes = false;                          // ← le champ courant a-t-il une section entre guillemets ?

    const pushField = () => {                            // ← point d'écriture unique
        row.push(fieldHadQuotes ? current : current.trim());  // ← cité = tel quel ; sinon on nettoie
        current = '';
        fieldHadQuotes = false;
    };

    for (let i = 0; i < text.length; i++) {
        const char = text[i];
        const next = text[i + 1];

        if (char === '"') {
            if (insideQuotes && next === '"') {
                current += '"';      // guillemet échappé ""
                i++;
            } else {
                insideQuotes = !insideQuotes;
                fieldHadQuotes = true;                  // ← on a vu au moins un guillemet
            }
        }
        else if (char === separator && !insideQuotes) {
            pushField();                                // ← remplace row.push(current.trim()); current = '';
        }
        else if ((char === '\n' || char === '\r') && !insideQuotes) {
            if (char === '\r' && next === '\n') i++;
            pushField();                                // ← idem
            rows.push(row);
            row = [];
        }
        else {
            current += char;
        }
    }

    if (current !== '' || row.length > 0) {
        pushField();                                    // ← idem
        rows.push(row);
    }

    return rows;
}
async function handleCompetenceData(codeEpci, type, fetchId) {
    try {

 const response = await fetchWithTimeout(`https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/${type.toLowerCase()}`, { method: 'GET' });
		
        if (!response.ok) {
            throw new Error(`Erreur réseau : ${response.status} ${response.statusText}`);
        }
        const text = await response.text();
        const rows = parseCsv(text, ',');
        const row = rows.find(r => r[0] === String(codeEpci));
        if (row) {
            const message = row[1] === "0" ? "non"
                          : row[1] === "1" ? "oui"
                          : "Valeur inconnue";
            setTextIfCurrent(fetchId,`competence${type}`, message);
        } else {
           setTextIfCurrent(fetchId,`competence${type}`, "Information non disponible");
        }
} catch (error) {
    console.error(`Erreur lors de la récupération des données ${type} :`, error);
    setTextIfCurrent(fetchId,`competence${type}`, "Information non disponible");
}
}


function handlePopulationData(data, fetchId) {

if (!Array.isArray(data) || data.length === 0 || typeof data[0] !== 'object' || typeof data[0].population !== 'number') {
        setTextIfCurrent(fetchId, 'populationInfo', 'Données non disponibles');
        return;
    }


    const population = data[0].population;
    if (Number.isInteger(population) && population >= 0 && population <= 100000000) {
        setTextIfCurrent(fetchId,'populationInfo', `${population} habitants`);
    } else {
        setTextIfCurrent(fetchId, 'populationInfo', 'Données non disponibles');
    }
}



async function handleEpciData(data, csvUrlPresident, fetchId) {
    if (!Array.isArray(data) || data.length === 0 || typeof data[0] !== 'object' || !data[0].epci || typeof data[0].epci.nom !== 'string' || typeof data[0].codeEpci !== 'string') {
        setTextIfCurrent(fetchId, 'epciInfo', 'Données non disponibles');
        return;
    }
    const epci = data[0].epci;
    const nomEpci = epci.nom || 'Non disponible';
    const codeEpci = data[0].codeEpci;

    if (codeEpci === SIREN_MGP) {
        setTextIfCurrent(fetchId, 'epciInfo', `Métropole du Grand Paris – dépend d'un EPT`);
    } else if (codeEpci) {
        setTextIfCurrent(fetchId, 'epciInfo', `${nomEpci} – (SIREN : ${codeEpci})`);
        await Promise.all([
            fetchAdresse(codeEpci, "epci", fetchId),
            fetchNomEluOuPresident("president", codeEpci, csvUrlPresident, fetchId)
        ]);
    } else {
        setTextIfCurrent(fetchId, 'epciInfo', 'Données non disponibles');
    }
}



async function handleMaireData(codeCommune, csvUrlMaire, fetchId) {
    await Promise.all([
        fetchNomEluOuPresident("maire", codeCommune, csvUrlMaire, fetchId),
        fetchAdresse(codeCommune, "mairie", fetchId)
    ]);
}

async function handleUniteUrbaineData(codeCommune, fetchId) {
    try {
        // ✅ Les deux téléchargements démarrent en même temps
const [inseeResponse, uuResponse] = await Promise.all([
            fetchWithTimeout('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/insee'),
            fetchWithTimeout('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/uu')
        ]);

        if (!inseeResponse.ok) throw new Error(`Erreur réseau (insee) : ${inseeResponse.status}`);
        if (!uuResponse.ok)   throw new Error(`Erreur réseau (uu) : ${uuResponse.status}`);

        // ✅ Les deux lectures .text() aussi en parallèle
        const [inseeText, uuText] = await Promise.all([
            inseeResponse.text(),
            uuResponse.text()
        ]);

        // Le traitement croisé reste séquentiel — c'est inévitable
 const inseeRows = parseCsv(inseeText, ',');
        const inseeLine = inseeRows.find(r => r[0] === String(codeCommune));

        if (!inseeLine) {
            setTextIfCurrent(fetchId,'popUrbaineInfo', "Information non disponible");
            return;
        }

       
		const numUniteUrbaine = inseeLine[1].substring(0, 5);

const uuRow = parseCsv(uuText, ',').find(r => r[0] === numUniteUrbaine);
        const numAssocie = uuRow ? parseInt(uuRow[1], 10) : null;

        if (numAssocie === null) {
            setTextIfCurrent(fetchId,'popUrbaineInfo', "hors unité urbaine");
            return;
        }

// numAssocie est la tranche d'unité urbaine (TUU) de l'INSEE :
        // un entier croissant avec la taille de l'agglomération.
        const TUU_MIN_100K = 6;  // 6 = premier palier ≥ 100 000 hab.
        const TUU_PARIS    = 8;  // 8 = agglomération de Paris

        const message =
            numAssocie < TUU_MIN_100K  ? "inférieure à 100000 habitants" :
            numAssocie < TUU_PARIS     ? "supérieure à 100000 habitants" :
            numAssocie === TUU_PARIS   ? "unité urbaine de Paris" :
                                         "Aucune condition spécifiée";

		setTextIfCurrent(fetchId,'popUrbaineInfo', message);

} catch (error) {
        console.error("Erreur unité urbaine :", error);
        setTextIfCurrent(fetchId,'popUrbaineInfo', "Information non disponible");
    }
}

async function handleSearch() {
    infosElement.textContent = '';

    // Cas 1 : une commune a déjà été choisie dans la liste
    if (selectedCodeCommune) {
        showLoading();
        await fetchData(selectedCodeCommune);
        return;
    }

    // Cas 2 : rien de sélectionné → on tente de résoudre le texte saisi
    const saisie = communeInput.value.trim();
    if (!validateInput(saisie, 50)) {
        showError("Veuillez entrer le nom d'une commune.");
        return;
    }

    showLoading();
    try {
        const url = `https://geo.api.gouv.fr/communes?nom=${encodeURIComponent(saisie)}`
                  + `&fields=nom,code,codeDepartement&limit=13`;
        const response = await fetchWithTimeout(url);
        if (!response.ok) throw new Error(`Erreur réseau : ${response.status}`);
        const data = await response.json();
        if (!Array.isArray(data)) throw new Error("Réponse de l'API invalide.");

        // correspondances EXACTES de nom, accents et casse ignorés
        const norm = s => String(s).normalize('NFD')
            .replace(/[\u0300-\u036f]/g, '').toLowerCase().trim();
        const exacts = data.filter(c => norm(c.nom) === norm(saisie));

        if (exacts.length === 1) {
            selectionnerCommune(exacts[0]);   // remplit l'en-tête, vide les cellules
            await fetchData(exacts[0].code);  // son finally appelle hideLoading()
        } else {
            // 0 ou plusieurs homonymes → on laisse l'utilisateur choisir
            hideLoading();
            fetchCommunes(saisie);
        }
    } catch (error) {
        console.error("Erreur lors de la résolution de la commune :", error);
        hideLoading();
        showError();
    }
}


function showError(userMessage = "Une erreur s'est produite. Veuillez réessayer plus tard.") {
    infosElement.textContent = userMessage;
}

function hideCommuneList() {
    communeList.innerHTML = '';
    communeList.style.display = 'none';
    communeInput.setAttribute('aria-expanded', 'false');
    communeInput.setAttribute('aria-activedescendant', '');
    activeIndex = -1;
}

function showCommuneList() {
    communeList.style.display = 'block';
    communeInput.setAttribute('aria-expanded', 'true');
}

function debounce(func, delay) {
    let debounceTimer;
    return function() {
        const context = this;
        const args = arguments;
        clearTimeout(debounceTimer);
        debounceTimer = setTimeout(() => func.apply(context, args), delay);
    };
}

const debouncedFetchCommunes = debounce(function(communeName) {
   if (!validateInput(communeName, 50)) {
        showError();
        hideCommuneList();
        return;
    }
    if (communeName.length >= 1) {
        fetchCommunes(communeName);
    } else {
        hideCommuneList();
    }
}, 300);

communeInput.addEventListener("input", function() {
    selectedCodeCommune = null;
    activeIndex = -1;
    debouncedFetchCommunes(this.value);
});

communeInput.addEventListener('keydown', function(e) {
    const items = communeList.querySelectorAll('li[role="option"]');
    if (!items.length) return;

    if (e.key === 'ArrowDown') {
        e.preventDefault();
        activeIndex = (activeIndex + 1) % items.length;
        updateFocus(items);
    } else if (e.key === 'ArrowUp') {
        e.preventDefault();
        activeIndex = (activeIndex - 1 + items.length) % items.length;
        updateFocus(items);
    } else if (e.key === 'Enter' && activeIndex >= 0) {
        e.preventDefault();
        items[activeIndex].click();
    } else if (e.key === 'Escape') {
        hideCommuneList();
    }
});

function updateFocus(items) {
    items.forEach((item, i) => {
        const isActive = i === activeIndex;
        item.setAttribute('aria-selected', isActive ? 'true' : 'false');
        if (isActive) {
            communeInput.setAttribute('aria-activedescendant', item.id);
            item.scrollIntoView({ block: 'nearest' });
        }
    });
}




async function fetchCommunes(communeName) {
    try {
		if (communeController) {
            communeController.abort();
        }
        communeController = new AbortController();
		const response = await fetchWithTimeout(`https://geo.api.gouv.fr/communes?nom=${encodeURIComponent(communeName)}&limit=13`, { signal: communeController.signal });

        if (!response.ok) {
            throw new Error("Erreur réseau lors de la récupération des communes.");
        }
        const data = await response.json();

// Réponse non conforme = vraie erreur → catch → showError()
        if (!Array.isArray(data)) {
            throw new Error("Les données retournées par l'API sont invalides.");
        }

        communeList.innerHTML = '';

        // Aucune correspondance = cas normal, pas un échec
        if (data.length === 0) {
            const emptyItem = document.createElement("li");
            emptyItem.textContent = "Aucune commune trouvée";
            emptyItem.setAttribute('aria-disabled', 'true'); // informatif, non sélectionnable
            communeList.appendChild(emptyItem);
            showCommuneList();
            return;
        }

       data.forEach(function(commune, index) {
            if (typeof commune.nom !== 'string' || typeof commune.codeDepartement !== 'string' || typeof commune.code !== 'string') {
                console.warn("Données de la commune invalides : ", commune);
                return;
            }

            const listItem = document.createElement("li");
                listItem.setAttribute('role', 'option');
    listItem.setAttribute('id', `commune-option-${index}`);
    listItem.setAttribute('aria-selected', 'false');
			listItem.textContent = `${normalizeText(commune.nom)} (${normalizeText(commune.codeDepartement)})`;
    listItem.addEventListener("click", function() {   // ← ICI
        selectionnerCommune(commune);
    })
            communeList.appendChild(listItem);
        });
        showCommuneList();
    } catch (error) {
		if (error.name === 'AbortError' || error.name === 'TimeoutError') return;
        showError();
        console.error("Détails de l'erreur :", error);
    }
}


document.addEventListener("click", function(event) {
    if (!communeInput.contains(event.target) && !communeList.contains(event.target)) {
        hideCommuneList();
    }
});

rechercherBtn.addEventListener("click", handleSearch);

// Nouvelle fonction de validation centralisée
function validateInput(text, maxLength = 100) {
    if (typeof text !== 'string') return false;
    if (text.length < 1 || text.length > maxLength) return false;
    return /^[\p{L}\p{M}\s'\u2018\u2019-]+$/u.test(text);
}



function normalizeText(text) {
    return String(text)
        .normalize('NFC')         // normalisation Unicode — accents français
                                  // pouvant arriver sous deux formes distinctes
        .replace(/\u00A0/g, ' ') // espaces insécables → espaces normaux
                                  // fréquents dans les retours d'API
        .replace(/\s+/g, ' ')    // espaces multiples → espace unique
                                  // utile après concaténation des champs d'adresse
        .trim();
}


async function getLatestCsvUrls() {
    try {

		const response = await fetchWithTimeout("https://www.data.gouv.fr/api/1/datasets/repertoire-national-des-elus-1/");
        if (!response.ok) throw new Error(`Erreur réseau : ${response.status}`);
        const data = await response.json();
        if (!Array.isArray(data.resources))
            throw new Error("Format de réponse inattendu : resources absent ou invalide.");
        return {
            urlMaire:      data.resources.find(r => r.title.includes("maires"))?.url ?? null,
            urlPresident:  data.resources.find(r => r.title.includes("conseillers-communautaires"))?.url ?? null
        };
    } catch (error) {
        console.error("Erreur récupération URLs CSV :", error);
        return { urlMaire: null, urlPresident: null };
    }
}


/**
 * Les URLs sont récupérées dynamiquement via getLatestCsvUrl().
 * Format des URLs statiques (pour référence) :
 *   maires      : https://static.data.gouv.fr/resources/repertoire-national-des-elus-1/{date}/elus-maires-mai.csv
 *   présidents  : https://static.data.gouv.fr/resources/repertoire-national-des-elus-1/{date}/elus-conseillers-communautaires-epci.csv
 */
async function fetchNomEluOuPresident(typeElu, code, csvUrl, fetchId) {
    const infoId = typeElu === "maire" ? "nomdumaire" : "nomdupresident";
    const rows = await fetchCsvData(csvUrl);

    if (!rows || rows.length < 2) {
        setTextIfCurrent(fetchId,infoId, "Information non disponible");
        return;
    }

    const header = rows[0];
    const data = rows.slice(1);

    // normalise un intitulé : retire accents, apostrophes, °, espaces, BOM…
    const norm = s => String(s)
        .normalize('NFD').replace(/[\u0300-\u036f]/g, '')
        .replace(/[^a-z0-9]/gi, '').toLowerCase();
    const col = label => header.findIndex(h => norm(h) === norm(label));

    const idx = typeElu === "maire"
        ? { code: col("Code de la commune"),
            nom:  col("Nom de l'élu"),
            prenom: col("Prénom de l'élu"),
            sexe: col("Code sexe") }
        : { code: col("N° SIREN"),
            nom:  col("Nom de l'élu"),
            prenom: col("Prénom de l'élu"),
            sexe: col("Code sexe"),
            fonction: col("Libellé de la fonction") };

    // une colonne attendue a disparu ou changé de nom → on le signale au lieu d'afficher "undefined"
    if (Object.values(idx).some(i => i === -1)) {
        console.warn("Colonne CSV introuvable. En-tête réel :", header);
       setTextIfCurrent(fetchId,infoId, "Information non disponible");
        return;
    }

    const codeRecherche = normalizeCode(code);

    for (const row of data) {
        const correspondanceCode = normalizeCode(row[idx.code]) === codeRecherche;
let correspondanceFonction = typeElu === "maire";
        if (!correspondanceFonction) {
            const f = norm(row[idx.fonction]);
            correspondanceFonction = f.startsWith("president") && f.includes("communautaire");
        }

        if (correspondanceCode && correspondanceFonction) {
            const sexeElu = row[idx.sexe] === "M" ? "M."
                          : row[idx.sexe] === "F" ? "Mme"
                          : "";
setTextIfCurrent(fetchId,infoId,
                `${sexeElu} ${row[idx.prenom]} ${row[idx.nom]}`);
            return;
        }
    }

    console.warn("Aucun élu correspondant trouvé pour le code :", code);
    setTextIfCurrent(fetchId,infoId, "Information non disponible");
}

async function fetchAdresse(code, type, fetchId) {
    const isMairie = type === 'mairie';
const whereClause = isMairie
        ? `pivot LIKE '%mairie%' AND pivot LIKE '%${code}%'`
        : `siren:"${code}"`;

    const params = new URLSearchParams({
        select: 'pivot,site_internet,nom,adresse_courriel,adresse',
        where: whereClause,
        limit: '100'
    });

    const apiUrl = `https://api-lannuaire.service-public.fr/api/explore/v2.1/catalog/datasets/api-lannuaire-administration/records?${params}`;

    try {

const response = await fetchWithTimeout(apiUrl, { method: 'GET' });
        if (!response.ok) {
            throw new Error(`Erreur réseau : ${response.status} ${response.statusText}`);
        }
        const data = await response.json();

        if (!Array.isArray(data.results) || data.results.length === 0) {
            throw new Error("Données d'adresse non disponibles ou format inattendu.");
        }

const records = data.results.filter(record => {
    const pivotData = safeJsonParse(record.pivot, []);
    return isMairie
        ? pivotData.some(item =>
            item.type_service_local === "mairie" &&
            Array.isArray(item.code_insee_commune) &&
            item.code_insee_commune.includes(code))
        : pivotData.some(item => item.type_service_local === "epci");
});
const record = records.find(r => r.nom.startsWith("Mairie - ")) || records[0];

        if (record && record.adresse) {
            const adresseData = safeJsonParse(record.adresse, []);
            if (!adresseData.length) throw new Error("Adresse JSON invalide.");
            const adresseComplete = [
                adresseData[0].numero_voie || '',
                adresseData[0].complement1 || '',
                adresseData[0].complement2 || '',
                adresseData[0].service_distribution || '',
                adresseData[0].code_postal || '',
                adresseData[0].nom_commune || ''
            ].filter(Boolean).join(' - ');

            if (adresseComplete) {
                const infoText = isMairie ? "adressemairie" : "adresseEpci";
                setTextIfCurrent(fetchId,infoText, adresseComplete);
            } else {
                console.warn("Adresse vide ou non valide :", adresseComplete);
            }

            if (record.adresse_courriel) {
                const infoText = isMairie ? "courrielmairie" : "courrielEpci";
                setTextIfCurrent(fetchId,infoText, record.adresse_courriel);
            }

            const siteInternetJSON = record.site_internet;
            if (siteInternetJSON) {
const siteInternetData = safeJsonParse(siteInternetJSON, []);
const siteInternet = siteInternetData.length > 0 ? siteInternetData[0].valeur : '';
                const infoText = isMairie ? "sitemairie" : "siteEpci";
                if (siteInternet && fetchId === latestFetchId) {
const anchorElement = document.createElement("a");
if (/^https?:\/\//i.test(siteInternet)) {
    anchorElement.href = siteInternet;
}
anchorElement.textContent = siteInternet;
anchorElement.target = "_blank";
anchorElement.rel = "noopener noreferrer";
document.getElementById(infoText).textContent = '';
document.getElementById(infoText).appendChild(anchorElement);

                }
            }
        } else {
            throw new Error("Aucune information sur la Mairie ou l'EPCI trouvée.");
        }

} catch (error) {
    console.error("Erreur lors de la récupération des données :", error);
    const ids = type === "mairie"
        ? { adresse: "adressemairie", courriel: "courrielmairie", site: "sitemairie" }
        : { adresse: "adresseEpci",   courriel: "courrielEpci",   site: "siteEpci"   };
setTextIfCurrent(fetchId,ids.adresse,  "Information non disponible");
setTextIfCurrent(fetchId,ids.courriel, "Information non disponible");
setTextIfCurrent(fetchId,ids.site,     "Information non disponible");
}
}


function validateApiResponse(data, expectedFields) {
    return expectedFields.every(field => field in data);
}

async function fetchData(selectedCodeCommune) {
    const fetchId = ++latestFetchId;
    const apiUrl = `https://geo.api.gouv.fr/communes?code=${selectedCodeCommune}&fields=code,population,codeEpci,epci`;

    try {
        const response = await fetchWithTimeout(apiUrl);
        if (!response.ok) {
            throw new Error(`Erreur réseau : ${response.status} ${response.statusText}`);
        }
        const data = await response.json();
        if (fetchId !== latestFetchId) return;

        if (data.length > 0 && validateApiResponse(data[0], ['code', 'population', 'epci'])) {
            const codeCommune = data[0].code;
            const codeEpci = data[0].codeEpci;

            const { urlMaire: csvUrlMaire, urlPresident: csvUrlPresident } = await getLatestCsvUrls();
            if (fetchId !== latestFetchId) return;
            console.log("URL maire :", csvUrlMaire);

            if (!csvUrlMaire)     setTextIfCurrent(fetchId, "nomdumaire", "Information non disponible");
            if (!csvUrlPresident) setTextIfCurrent(fetchId, "nomdupresident", "Information non disponible");

            const matchDate = csvUrlMaire && csvUrlMaire.match(/\/(\d{4})(\d{2})(\d{2})-\d{6}\//);
            if (matchDate) {
                setTextIfCurrent(fetchId, 'sourceRNEDate',
                    `(mise à jour du ${matchDate[3]}/${matchDate[2]}/${matchDate[1]})`);
            }

            await Promise.all([
                handlePopulationData(data, fetchId),
                handleEpciData(data, csvUrlPresident, fetchId),
                handleMaireData(codeCommune, csvUrlMaire, fetchId),
                handleUniteUrbaineData(codeCommune, fetchId),
                codeEpci ? handleCompetenceData(codeEpci, 'PLU', fetchId) : Promise.resolve(),
                codeEpci ? handleCompetenceData(codeEpci, 'RLP', fetchId) : Promise.resolve()
            ]);

            if (fetchId !== latestFetchId) return;
            document.querySelectorAll("table").forEach(table => {
                table.style.display = "table";
            });
        } else {
            showError();
        }
    } catch (error) {
        console.error("Une erreur s'est produite lors de la récupération des données de l'API :", error);
        if (fetchId === latestFetchId) showError();
    } finally {
        if (fetchId === latestFetchId) hideLoading();
    }
}

	});
	</script>


	<hr> <b>Sources :</b>
	<ul style="list-style-type:square">
		<li>(1) API gouvernementale : <a href="https://geo.api.gouv.fr/decoupage-administratif/communes" target="_blank" rel="noopener noreferrer">https://geo.api.gouv.fr/decoupage-administratif/communes</a></li>
		<li>(2) informations mises à jour manuellement – valable au 1er janvier 2026 – source : <a href="https://www.insee.fr/fr/information/4802589" target="_blank" rel="noopener noreferrer">https://www.insee.fr/fr/information/4802589</a></li>
		<li>(3) OpenData gouvernemental : Ministère de l'Intérieur et des Outre-Mer – <a href="https://www.data.gouv.fr/fr/datasets/repertoire-national-des-elus-1/" target="_blank" rel="noopener noreferrer">https://www.data.gouv.fr/fr/datasets/repertoire-national-des-elus-1/</a> <span id="sourceRNEDate"></span></li>
		<li>(4) API gouvernementale : <a href="https://api-lannuaire.service-public.fr/explore/dataset/api-lannuaire-administration" target="_blank" rel="noopener noreferrer">https://api-lannuaire.service-public.fr/explore/dataset/api-lannuaire-administration</a></li>
		<li>(5) informations mises à jour manuellement (intercommunalité puis Export national ou régional) – valable au 22 mars 2026 – source : <a href="https://www.banatic.interieur.gouv.fr/export/" target="_blank" rel="noopener noreferrer">https://www.banatic.interieur.gouv.fr/export/</a></li>
	</ul>

	<hr> <b>Historique :</b>
	<ul style="list-style-type:square">
		<li>version 1.36c du 22/06/2026 : Mise à jour du code</li>
		<li>version 1.35s du 21/06/2026 : Mise à jour du code</li>
		<li>version 1.34e du 20/06/2026 : Mise à jour du code</li>
		<li>version 1.33p du 19/06/2026 : Mise à jour du code</li>
	    <li>version 1.32c du 18/06/2026 : Mise à jour du code</li>
	    <li>version 1.31f du 15/06/2026 : Mise à jour du code</li>
	    <li>version 1.30ad du 14/06/2026 : Mise à jour du code</li>
		<li>version 1.29t du 10/05/2026 : Correctif + Mise à jour des fichiers des noms des maires et présidents d'EPCI</li>
	    <li>version 1.28b du 01/05/2026 : Mise à jour des fichiers des noms des maires et présidents d'EPCI</li>
	    <li>version 1.27c du 22/03/2026 : Mise à jour des fichiers des unités urbaines, des compétences PLU et RLP</li>
		<li>version 1.26a du 24/12/2025 : Mise à jour des fichiers des noms des maires et présidents d'EPCI</li>
		<li>version 1.25b du 14/10/2025 : Mise à jour des fichiers des noms des maires et présidents d'EPCI</li>
		<li>version 1.24b du 15/06/2025 : Mise à jour des fichiers des noms des maires et présidents d'EPCI. Optimisation du code</li>
 		<li>version 1.23d du 05/06/2025 : Mise à jour des fichiers des unités urbaines et des compétences PLU. Ajout de la compétence RLP</li>
		<li>version 1.22c du 16/03/2025 : Mise à jour des fichiers des noms des maires et présidents d'EPCI</li>
 		<li>version 1.21j du 12/02/2025 : Résolution du problème avec les noms des maires en Corse + correction d'un bug sur les adresses des grandes villes + correction de l'affichage des apostrophes dans les adresses</li>
 		<li>version 1.20a du 11/02/2025 : Mise à jour des fichiers des noms des maires et présidents d'EPCI</li>
 		<li>version 1.19g du 27/10/2024 : Amélioration de la simplicité</li>
 		<li>version 1.18t du 26/10/2024 : Amélioration de la sécurité</li>
 		<li>version 1.17b du 24/10/2024 : Amélioration de la sécurité</li>
 		<li>version 1.16g du 21/10/2024 : Amélioration de la sécurité</li>
   		<li>version 1.15m du 20/10/2024 : Amélioration de la sécurité</li>
 		<li>version 1.14u du 19/10/2024 : Amélioration de la sécurité</li>
		<li>version 1.13h du 18/10/2024 : Amélioration de la sécurité</li>
  		<li>version 1.12f du 17/10/2024 : Amélioration de la sécurité</li>
 		<li>version 1.11g du 03/09/2024 : Résolution d'un bug - suppression de l'integrity de Axios</li>
 		<li>version 1.10c du 01/09/2024 : Modification de integrity de Axios suite à mise à jour (1.7.7) et de jQuery</li>
		<li>version 1.09b du 25/08/2024 : Modification de integrity de Axios suite à mise à jour (1.7.5)</li>
  		<li>version 1.08c du 06/08/2024 : Modification de integrity de Axios suite à mise à jour (1.7.3), remplacement de csvUrlMaire et de csvUrlPresident, mise à jour de la source (5)</li>
		<li>version 1.07c du 03/06/2024 : suppression de la balise meta http-equiv="X-Frame-Options" content="SAMEORIGIN", modification de integrity de Axios, ajout de Axios dans la liste des librairies</li>
 		<li>version 1.06c du 22/03/2024 : Mise à jour du CSP</li>
		<li>version 1.05a du 18/03/2024 : Mise à jour des bases de données compétence PLU et unité urbaine</li>
 		<li>version 1.04t du 17/03/2024 : Implantation Content Security Policy header, Subresource integrity, X-Content-Type-Options header, X-Frame-Options header, referrer-policy header</li>
		<li>version 1.03a du 14/01/2024 : Suppression du style imposé par Github Pages</li>
 		<li>version 1.02a du 13/01/2024 : Migration du code principal vers Github et adaptation</li>
		<li>version 1.01a du 11/01/2024 : Ajout des liens hypertextes</li>
		<li>version 1.0a du 01/01/2024 : Mise en ligne</li>
	</ul>
	<hr>
 <script>
document.addEventListener('DOMContentLoaded', function() {
  var githubLink = document.querySelector('h1 a[href="https://paysagesdefrance.github.io/"]');
  var unwantedStyle = document.querySelector('link[href^="/assets/css/style.css"]');
  if (githubLink) {
    githubLink.parentElement.style.display = 'none';
  }
  if (unwantedStyle) {
    unwantedStyle.parentNode.removeChild(unwantedStyle);
  }});
</script>
 </body>
</html>
