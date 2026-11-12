<html lang="fr">

<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<link rel="preconnect" href="https://geo.api.gouv.fr" crossorigin>
	<link rel="dns-prefetch" href="https://www.data.gouv.fr">
	<link rel="dns-prefetch" href="https://static.data.gouv.fr">
	<link rel="dns-prefetch" href="https://api-lannuaire.service-public.fr">
	<link rel="dns-prefetch" href="https://raw.githubusercontent.com">
	<link rel="dns-prefetch" href="https://cdn.jsdelivr.net">

	<!-- Anti-framing réel = frame-buster JS plus bas (#af + script).
	     frame-ancestors ci-dessous est ignoré en mode <meta> (spec CSP),
	     conservé uniquement pour portabilité vers un en-tête HTTP réel.
	     'self' retiré de script-src et connect-src : la page n'utilise aucun
	     script externe ni appel vers sa propre origine (*.github.io est
	     partagée entre tous les dépôts du compte). -->
	<meta http-equiv="Content-Security-Policy" content="default-src 'none';
		script-src 'unsafe-inline';
		style-src 'unsafe-inline';
		connect-src https://geo.api.gouv.fr
			https://api-lannuaire.service-public.fr
			https://raw.githubusercontent.com
			https://static.data.gouv.fr
			https://www.data.gouv.fr
			https://cdn.jsdelivr.net;
		base-uri 'none';
		form-action 'none';
		frame-ancestors 'none';">

	<!-- Vie privée : aucun Referer envoyé aux API ni aux sites ouverts depuis la page. -->
	<meta name="referrer" content="no-referrer">

	<title>Recherche d'une commune</title>

	<style id="af">html { display: none !important; }</style>
	<noscript>
		<style>
			/* Appliqué uniquement si JS est désactivé : annule #af pour ne pas
			   laisser un écran blanc, et masque l'appli non fonctionnelle. */
			html { display: block !important; }
			body > :not(.js-requise) { display: none !important; }
			.js-requise { display: block !important; margin: 2em; }
		</style>
	</noscript>
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

	.sr-only {
		position: absolute;
		width: 1px; height: 1px;
		margin: -1px; padding: 0; border: 0;
		overflow: hidden;
		clip: rect(0, 0, 0, 0);
		clip-path: inset(50%);
		white-space: nowrap;
	}
	.js-requise { display: none; }

	.combobox {
		position: relative;
		display: inline-block;
		width: 200px;
		margin-right: 20px;
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
		max-height: 40vh;
		overflow-y: auto;
	}

	.combobox .dropdown-menu li {
		padding: 5px;
		cursor: pointer;
	}

	.combobox .dropdown-menu li:hover {
		background-color: #ddd;
	}
	.combobox .dropdown-menu li[aria-disabled="true"] {
		cursor: default;
		color: #666;
	}
	.combobox .dropdown-menu li[aria-disabled="true"]:hover {
		background-color: transparent;
	}

	.aide {
		font-size: 0.85em;
		color: #555;
	}

	.error-message {
		color: red;
	}

	#infos {
		color: #b00020;
		margin-top: 8px;
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

	/* Tableau d'état des services : toujours visible (contrairement aux résultats) */
	#etatServices { display: table; margin-top: 10px; }
	.etat-ok      { color: #1b5e20; }
	.etat-lent    { color: #8a5a00; }
	.etat-ko      { color: #b00020; }
	.etat-attente,
	.etat-inconnu { color: #555; }
	#etatResume   { margin: 8px 0; }

	@media (prefers-reduced-motion: reduce) {
		.spinner { animation-duration: 2.5s; }
	}
	</style>
</head>

<body>
	<p class="js-requise">
		Cette application nécessite JavaScript pour fonctionner.
		Merci de l'activer dans les paramètres de votre navigateur, puis de recharger la page.
	</p>
	<h1>Recherche d'une commune</h1>
	<div class="combobox">
		<label for="communeInput">Commune ou code postal</label>
		<input
			type="text"
			id="communeInput"
			name="commune"
			autocomplete="off"
			spellcheck="false"
			maxlength="100"
			enterkeyhint="search"
			role="combobox"
			aria-expanded="false"
			aria-controls="commune-list"
			aria-autocomplete="list"
			aria-haspopup="listbox"
			aria-describedby="commune-aide"
			aria-activedescendant="">
		<ul
			id="commune-list"
			class="dropdown-menu"
			role="listbox"
			aria-label="Communes suggérées">
		</ul>
	</div>
	<div id="commune-status" role="status" aria-live="polite" class="sr-only"></div>
	<button type="button" id="rechercherBtn">Rechercher</button>
	<p id="commune-aide" class="aide">Saisissez un nom de commune ou un code postal à 5 chiffres.</p>
	<div id="chargement" role="status" aria-live="polite" aria-busy="false">
		<span class="spinner" aria-hidden="true"></span>
		Chargement en cours…
	</div>
	<div id="resultatCommune" aria-live="polite"></div>
	<div id="infos" role="alert"></div>
	<table class="resultat">
		<tr>
			<th colspan="2" scope="col"><b>– Population –</b></th>
		</tr>
		<tr>
			<th scope="row">Population de la commune <sup>(1)</sup></th>
			<td id="populationInfo"></td>
		</tr>
		<tr>
			<th scope="row">Population de l'unité urbaine <sup>(2)</sup></th>
			<td id="popUrbaineInfo"></td>
		</tr>
	</table>
	<br>
	<br>
	<table class="resultat">
		<tr>
			<th colspan="2" scope="col"><b>– Mairie –</b></th>
		</tr>
		<tr>
			<th scope="row">Nom du maire <sup>(3)</sup></th>
			<td id="nomdumaire"></td>
		</tr>
		<tr>
			<th scope="row">Adresse de la mairie <sup>(4)</sup></th>
			<td id="adressemairie"></td>
		</tr>
		<tr>
			<th scope="row">Téléphone de la mairie <sup>(4)</sup></th>
			<td id="telephonemairie"></td>
		</tr>
		<tr>
			<th scope="row">Adresse courriel de la mairie <sup>(4)</sup></th>
			<td id="courrielmairie"></td>
		</tr>
		<tr>
			<th scope="row">Site internet de la mairie <sup>(4)</sup></th>
			<td id="sitemairie"></td>
		</tr>
	</table>
	<br>
	<br>
	<table class="resultat">
		<tr>
			<th colspan="2" scope="col"><b>– EPCI –</b></th>
		</tr>
		<tr>
			<th scope="row">EPCI <sup>(1)</sup></th>
			<td id="epciInfo"></td>
		</tr>
		<tr>
			<th scope="row">Nom du président <sup>(3)</sup></th>
			<td id="nomdupresident"></td>
		</tr>
		<tr>
			<th scope="row">Adresse de l'EPCI <sup>(4)</sup></th>
			<td id="adresseEpci"></td>
		</tr>
		<tr>
			<th scope="row">Téléphone de l'EPCI <sup>(4)</sup></th>
			<td id="telephoneEpci"></td>
		</tr>
		<tr>
			<th scope="row">Adresse courriel de l'EPCI <sup>(4)</sup></th>
			<td id="courrielEpci"></td>
		</tr>
		<tr>
			<th scope="row">Site internet de l'EPCI <sup>(4)</sup></th>
			<td id="siteEpci"></td>
		</tr>
		<tr>
			<th scope="row">Compétence PLU <sup>(5)</sup></th>
			<td id="competencePLU"></td>
		</tr>
		<tr>
			<th scope="row">Compétence RLP <sup>(5)</sup></th>
			<td id="competenceRLP"></td>
		</tr>
	</table>
	<br>
	<hr>
	<h2 id="etatTitre">État des services</h2>
	<p class="aide">Disponibilité des sources de données utilisées par cette page. L'état est mis à jour à chaque vérification et à chaque recherche.</p>
	<table id="etatServices" aria-labelledby="etatTitre">
		<tr>
			<th scope="col">Service</th>
			<th scope="col">Source</th>
			<th scope="col">État</th>
			<th scope="col">Dernière réponse</th>
		</tr>
		<tr>
			<th scope="row">Découpage administratif (geo.api.gouv.fr)</th>
			<td>(1)</td>
			<td id="etat-geo" class="etat-attente">non vérifié</td>
			<td id="heure-geo"></td>
		</tr>
		<tr>
			<th scope="row">Catalogue data.gouv.fr (liste des fichiers du RNE)</th>
			<td>(3)</td>
			<td id="etat-datagouv" class="etat-attente">non vérifié</td>
			<td id="heure-datagouv"></td>
		</tr>
		<tr>
			<th scope="row">Fichiers du Répertoire national des élus (static.data.gouv.fr)</th>
			<td>(3)</td>
			<td id="etat-rne" class="etat-attente">non vérifié</td>
			<td id="heure-rne"></td>
		</tr>
		<tr>
			<th scope="row">Annuaire de l'administration (service-public.fr)</th>
			<td>(4)</td>
			<td id="etat-annuaire" class="etat-attente">non vérifié</td>
			<td id="heure-annuaire"></td>
		</tr>
		<tr>
			<th scope="row">Fichiers unités urbaines et compétences (GitHub)</th>
			<td>(2) (5)</td>
			<td id="etat-github" class="etat-attente">non vérifié</td>
			<td id="heure-github"></td>
		</tr>
		<tr>
			<th scope="row">Liste des EPT de la Métropole du Grand Paris (Etalab, via jsDelivr)</th>
			<td>(1)</td>
			<td id="etat-etalab" class="etat-attente">non vérifié</td>
			<td id="heure-etalab"></td>
		</tr>
	</table>
	<p id="etatResume" role="status" aria-live="polite"></p>
	<button type="button" id="verifierServicesBtn">Vérifier les services</button>
	<br>
	<script>
	document.addEventListener('DOMContentLoaded', function() {
	'use strict';

	const communeInput = document.getElementById("communeInput");
	const communeList = document.getElementById("commune-list");
	const rechercherBtn = document.getElementById("rechercherBtn");
	const infosElement = document.getElementById("infos");
	const communeStatus = document.getElementById("commune-status");
	const comboboxEl = communeInput.closest('.combobox');

	let selectedCodeCommune = null;
	let activeIndex = -1;
	let communeController = null;
	let lastCommuneResult = null;   // { query, data } du dernier rendu d'autocomplétion
	const csvCache = {};
	const CSV_CACHE_TTL = 10 * 60 * 1000; // 10 min, en ms
	const csvInFlight = {};   // url -> Promise en cours (dédoublonnage des téléchargements concurrents)
	let csvUrlsCache = null;              // { value, timestamp } — null tant qu'aucun succès
	let csvUrlsInFlight = null;           // Promise en cours (dédoublonnage des appels concurrents)
	const SIREN_MGP = "200054781";
	const CODE_PARIS = "75056";

	// Établissements publics territoriaux (EPT) de la Métropole du Grand Paris.
	// geo.api.gouv.fr ne fournit pas le rattachement commune -> EPT : il est lu en direct
	// dans le fichier officiel ept.json du « découpage administratif » d'Etalab (source
	// de geo.api.gouv.fr), diffusé par le paquet npm @etalab/decoupage-administratif et
	// servi par jsDelivr dans sa dernière version publiée. Aucune donnée figée ici.
	// Président et coordonnées des EPT : ni le RNE ni l'Annuaire ne les publient, et
	// l'API BANATIC refuse les appels depuis un navigateur (CORS) → lien vers la fiche
	// BANATIC officielle, toujours à jour.
	const URL_EPT = 'https://cdn.jsdelivr.net/npm/@etalab/decoupage-administratif/data/ept.json';
	let eptCache = null;      // { index: Map(code INSEE -> { siren, nom }), timestamp }
	let eptInFlight = null;

	const urlFicheBanatic = siren => `https://www.banatic.interieur.gouv.fr/intercommunalite/${siren}`;

	/** Index code INSEE -> EPT, ou null si le fichier est indisponible ou invalide. */
	async function chargerEpt() {
		if (eptCache && (Date.now() - eptCache.timestamp) < CSV_CACHE_TTL) return eptCache.index;
		if (eptInFlight) return eptInFlight;
		eptInFlight = (async () => {
			try {
				const response = await fetchWithTimeout(URL_EPT);
				if (!response.ok) throw new Error(`Erreur réseau : ${response.status}`);
				const liste = await response.json();
				if (!Array.isArray(liste)) throw new Error("ept.json : format inattendu.");
				const index = new Map();
				for (const e of liste) {
					// Contrôle strict : données externes
					if (!e || typeof e.code !== 'string' || !/^\d{9}$/.test(e.code)
						|| typeof e.nom !== 'string' || !e.nom || e.nom.length > 200
						|| !Array.isArray(e.membres)) continue;
					const ept = { siren: e.code, nom: e.nom };
					e.membres.forEach(m => {
						if (m && typeof m.code === 'string' && /^(?:\d{5}|2[AB]\d{3})$/.test(m.code)) index.set(m.code, ept);
					});
				}
				if (index.size === 0) throw new Error("ept.json : aucune entrée valide.");
				eptCache = { index, timestamp: Date.now() };
				return index;
			} catch (error) {
				console.error("Erreur chargement des EPT :", error);
				return null;
			} finally {
				eptInFlight = null;
			}
		})();
		return eptInFlight;
	}

	/** EPT de rattachement : { siren, nom }, { indisponible: true } si la liste est inaccessible, ou null. */
	async function eptDeCommune(codeCommune) {
		const index = await chargerEpt();
		if (!index) return { indisponible: true };
		return index.get(codeCommune) || null;
	}

	let latestFetchId = 0;

	// Formats de codes acceptés
	const RE_INSEE = /^(?:\d{5}|2[AB]\d{3})$/;
	const RE_CP = /^\d{5}$/;
	const RE_CP_PARTIEL = /^\d{1,4}$/;
	const RE_NOM = /^[\p{L}\p{M}\s'‘’-]+$/u;

	// Hôtes autorisés pour les CSV du RNE (doit rester cohérent avec connect-src du CSP).
	const HOTES_RNE = ['static.data.gouv.fr', 'www.data.gouv.fr'];

	const CELL_IDS = ['populationInfo','popUrbaineInfo','epciInfo','nomdumaire',
		'adressemairie','telephonemairie','courrielmairie','sitemairie','nomdupresident',
		'adresseEpci','telephoneEpci','courrielEpci','siteEpci','competencePLU','competenceRLP'];

	// Partagée entre l'autocomplétion (fetchCommunes) et la résolution directe
	// (handleSearch). Les deux DOIVENT rester identiques : lastCommuneResult est
	// réutilisé de l'une vers l'autre, comparer des résultats obtenus avec des
	// paramètres différents serait incohérent.
	// boost=population : les grandes communes remontent en tête des suggestions.
	const buildCommuneSearchUrl = q => RE_CP.test(q)
		? `https://geo.api.gouv.fr/communes?codePostal=${q}&fields=nom,code,codeDepartement`
		: `https://geo.api.gouv.fr/communes?nom=${encodeURIComponent(q)}`
		+ `&fields=nom,code,codeDepartement&boost=population&limit=13`;

	/** Une commune renvoyée par l'API est exploitable (types + format du code INSEE). */
	function communeValide(c) {
		return c !== null && typeof c === 'object'
			&& typeof c.nom === 'string'
			&& typeof c.codeDepartement === 'string'
			&& typeof c.code === 'string'
			&& RE_INSEE.test(c.code);
	}

	function clearCells() {
		CELL_IDS.forEach(id => { document.getElementById(id).textContent = ''; });
		// Annotation de source (hors tableaux) : évite de conserver la date de la
		// recherche précédente si la nouvelle ne parvient pas à la déterminer.
		document.getElementById('sourceRNEDate').textContent = '';
	}

	function renderCommuneList(data) {
		communeList.innerHTML = '';
		activeIndex = -1;
		communeInput.setAttribute('aria-activedescendant', '');

		const valides = data.filter(c => {
			if (communeValide(c)) return true;
			console.warn("Données de la commune invalides : ", c);
			return false;
		});

		if (valides.length === 0) {
			communeStatus.textContent = "Aucune commune trouvée";
			const emptyItem = document.createElement("li");
			emptyItem.textContent = "Aucune commune trouvée";
			emptyItem.setAttribute('role', 'option');
			emptyItem.setAttribute('aria-disabled', 'true');
			communeList.appendChild(emptyItem);
			showCommuneList();
			return;
		}

		valides.forEach(function(commune, index) {
			const listItem = document.createElement("li");
			listItem.setAttribute('role', 'option');
			listItem.setAttribute('id', `commune-option-${index}`);
			listItem.setAttribute('aria-selected', 'false');
			listItem.textContent = `${normalizeText(commune.nom)} (${normalizeText(commune.codeDepartement)})`;
			listItem.addEventListener("click", function() {
				choisirCommune(commune);
			});
			communeList.appendChild(listItem);
		});
		const n = valides.length;
		communeStatus.textContent = `${n} commune${n > 1 ? 's' : ''} trouvée${n > 1 ? 's' : ''}`;
		showCommuneList();
	}

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

	function setNodeIfCurrent(fetchId, elementId, node) {
		if (fetchId !== latestFetchId) return;
		const element = document.getElementById(elementId);
		if (!element) {
			console.warn(`Élément introuvable : ${elementId}`);
			return;
		}
		element.textContent = '';
		element.appendChild(node);
	}

	/** Affiche un lien (href déjà validé par l'appelant) ou, à défaut, le texte seul. */
	function setLienIfCurrent(fetchId, elementId, href, texte, nouvelOnglet) {
		if (!href) {
			setTextIfCurrent(fetchId, elementId, texte);
			return;
		}
		const a = document.createElement('a');
		a.href = href;
		a.textContent = normalizeText(texte);
		if (nouvelOnglet) {
			a.target = '_blank';
			a.rel = 'noopener noreferrer';
		}
		setNodeIfCurrent(fetchId, elementId, a);
	}

	/**
	 * Normalise un code INSEE ou SIREN pour la comparaison.
	 * Nécessaire car les sources sont incohérentes : l'API geo.gouv.fr
	 * retourne "01001" tandis que les CSV du RNE peuvent retourner "1001".
	 * Les codes corses (2A, 2B) ne commençant pas par zéro, ils ne sont pas affectés.
	 */
	function normalizeCode(c) {
		if (c == null) return '';
		return String(c).trim().toUpperCase().replace(/^0+/, '');
	}

	/** Pré-remplit l'interface pour la commune choisie (sans lancer la recherche). */
	function selectionnerCommune(commune) {
		selectedCodeCommune = commune.code;
		communeInput.value = normalizeText(commune.nom);
		hideCommuneList();
		infosElement.textContent = '';
		clearCells();
		afficherEnTete(commune);
	}

	function afficherEnTete(commune) {
		const resultatCommune = document.getElementById('resultatCommune');
		const h2Element = document.createElement('h2');
		h2Element.textContent = normalizeText(
			`– ${commune.nom} (${commune.codeDepartement}) – code INSEE ${commune.code}`);
		resultatCommune.textContent = '';
		resultatCommune.appendChild(h2Element);
	}

	/** Sélection depuis la liste (clic ou Entrée) : sélection + recherche immédiate. */
	function choisirCommune(commune) {
		selectionnerCommune(commune);
		lancerRecherche(commune.code);
	}

	/** Lance la récupération complète des données pour un code INSEE. */
	function lancerRecherche(code) {
		debouncedFetchCommunes.cancel();
		if (communeController) communeController.abort();
		// Vidage AVANT la re-recherche : en cas d'échec réseau, on affiche
		// le message d'erreur seul, pas superposé aux données périmées.
		document.querySelectorAll("table.resultat").forEach(t => { t.style.display = "none"; });
		clearCells();
		infosElement.textContent = '';
		showLoading();
		const fetchId = ++latestFetchId;
		return fetchData(code, fetchId);
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

	function resetAffichage() {
		document.querySelectorAll("table.resultat").forEach(t => { t.style.display = "none"; });
		document.getElementById('resultatCommune').textContent = '';
		infosElement.textContent = '';
		clearCells();
	}

	function safeJsonParse(value, fallback = null) {
		try {
			return JSON.parse(value);
		} catch (e) {
			console.warn("JSON invalide :", value, e);
			return fallback;
		}
	}

	/** Champ de l'Annuaire : chaîne JSON ou tableau déjà décodé → toujours un tableau. */
	function champJson(value) {
		if (Array.isArray(value)) return value;
		if (typeof value !== 'string' || value === '') return [];
		const parsed = safeJsonParse(value, []);
		return Array.isArray(parsed) ? parsed : [];
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

		const debut = performance.now();
		try {
			// credentials: 'omit' → aucun cookie envoyé aux API tierces (vie privée).
			const response = await fetch(url, {
				credentials: 'omit',
				referrerPolicy: 'no-referrer',
				...options,
				signal: controller.signal
			});
			enregistrerEtat(url, { status: response.status, ms: performance.now() - debut });
			return response;
		} catch (error) {
			// Un abandon volontaire (nouvelle saisie) ne dit rien de l'état du service.
			if (error.name !== 'AbortError') enregistrerEtat(url, { erreur: error });
			throw error;
		} finally {
			clearTimeout(timeoutId);
			if (options.signal) options.signal.removeEventListener('abort', onExternalAbort);
		}
	}

	async function fetchCsvData(url, separator = ';') {
		if (!url) return null;
		const cached = csvCache[url];
		if (cached && (Date.now() - cached.timestamp) < CSV_CACHE_TTL) {
			return cached.data;
		}
		// Téléchargement du même fichier déjà en cours (deux recherches rapprochées, ou
		// deux appelants du même CSV) → on réutilise la promesse au lieu d'ouvrir une 2e connexion.
		if (csvInFlight[url]) {
			return csvInFlight[url];
		}
		const promise = (async () => {
			try {
				// Fichiers volumineux (plusieurs Mo) : délai plus large que pour les API.
				const response = await fetchWithTimeout(url, {}, 30000);
				if (!response.ok) {
					throw new Error(`Erreur réseau : ${response.status}`);
				}
				let text = await response.text();
				if (text.charCodeAt(0) === 0xFEFF) text = text.slice(1);   // BOM UTF-8
				const data = parseCsv(text, separator);
				csvCache[url] = { data, timestamp: Date.now() };
				return data;
			} catch (error) {
				console.error(error);
				return null;
			} finally {
				delete csvInFlight[url];   // libère le verrou, succès comme échec
			}
		})();
		csvInFlight[url] = promise;
		return promise;
	}

	function parseCsv(text, separator = ';') {
		const physical = text.split(/\r\n|\r|\n/);
		// Un fichier terminé par un saut de ligne produit une dernière entrée
		// vide : on la retire pour ne pas générer une ligne fantôme.
		if (physical.length && physical[physical.length - 1] === '') physical.pop();

		const rows = [];
		let buffer = null;
		let quotes = 0;
		for (const line of physical) {
			buffer = buffer === null ? line : buffer + '\n' + line;
			// Un champ entre guillemets peut contenir un saut de ligne : tant que
			// le nombre de " est impair, l'enregistrement n'est pas terminé.
			// (compte incrémental : évite de recompter tout le tampon à chaque ligne)
			quotes += (line.match(/"/g) || []).length;
			if (quotes % 2 !== 0) continue;
			rows.push(parseCsvLine(buffer, separator));
			buffer = null;
			quotes = 0;
		}
		if (buffer !== null) rows.push(parseCsvLine(buffer, separator)); // guillemet non fermé
		return rows;
	}

	function parseCsvLine(line, separator) {
		// Chemin rapide : aucun guillemet → split natif
		if (line.indexOf('"') === -1) {
			return line.split(separator).map(f => f.trim());
		}
		// Chemin lent : machine à états appliquée à une seule ligne.
		// Champs cités préservés tels quels ; sinon trim — sémantique identique.
		const fields = [];
		let current = '';
		let insideQuotes = false;
		let fieldHadQuotes = false;
		for (let i = 0; i < line.length; i++) {
			const char = line[i];
			const next = line[i + 1];
			if (char === '"') {
				if (insideQuotes && next === '"') { current += '"'; i++; }
				else { insideQuotes = !insideQuotes; fieldHadQuotes = true; }
			} else if (char === separator && !insideQuotes) {
				fields.push(fieldHadQuotes ? current : current.trim());
				current = ''; fieldHadQuotes = false;
			} else {
				current += char;
			}
		}
		fields.push(fieldHadQuotes ? current : current.trim());
		return fields;
	}

	async function handleCompetenceData(codeEpci, type, fetchId, codeCommune) {
		if (codeEpci === SIREN_MGP) {
			// Paris, ou EPT de rattachement inconnu (liste des EPT inaccessible)
			setTextIfCurrent(fetchId, `competence${type}`, codeCommune === CODE_PARIS
				? "sans objet (compétence exercée par la Ville de Paris)"
				: "Information non disponible");
			return;
		}
		const url = `https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/${type.toLowerCase()}`;
		const rows = await fetchCsvData(url, ',');

		if (!rows) {
			setTextIfCurrent(fetchId, `competence${type}`, "Information non disponible");
			return;
		}

		const row = rows.find(r => normalizeCode(r[0]) === normalizeCode(codeEpci));
		if (row) {
			const message = row[1] === "0" ? "non"
				: row[1] === "1" ? "oui"
				: "Valeur inconnue";
			setTextIfCurrent(fetchId, `competence${type}`, message);
		} else {
			setTextIfCurrent(fetchId, `competence${type}`, "Information non disponible");
		}
	}

	function handlePopulationData(data, fetchId) {
		const population = Array.isArray(data) && data[0] ? data[0].population : undefined;
		if (Number.isInteger(population) && population >= 0 && population <= 100000000) {
			// 0 et 1 : singulier en français (« 0 habitant », « 1 habitant »).
			setTextIfCurrent(fetchId, 'populationInfo',
				`${population.toLocaleString('fr-FR')} habitant${population > 1 ? 's' : ''}`);
		} else {
			setTextIfCurrent(fetchId, 'populationInfo', 'Données non disponibles');
		}
	}

	function afficherCommuneSansEpci(fetchId) {
		setTextIfCurrent(fetchId, 'epciInfo', "Commune sans EPCI à fiscalité propre");
		['nomdupresident', 'adresseEpci', 'telephoneEpci', 'courrielEpci', 'siteEpci',
		 'competencePLU', 'competenceRLP']
			.forEach(id => setTextIfCurrent(fetchId, id, 'sans objet'));
	}

	async function handleEpciData(data, csvUrlPresident, fetchId, codeCommune, ept) {
		if (!Array.isArray(data) || data.length === 0 || typeof data[0] !== 'object' || !data[0].epci || typeof data[0].epci.nom !== 'string' || typeof data[0].codeEpci !== 'string') {
			setTextIfCurrent(fetchId, 'epciInfo', 'Données non disponibles');
			return;
		}
		const nomEpci = data[0].epci.nom || 'Non disponible';
		const codeEpci = data[0].codeEpci;

		if (ept && !ept.indisponible) {
			// Commune de la MGP : nom de l'EPT (compétences PLU / RLP de l'EPT : cf. fetchData).
			// Président et coordonnées : lien vers la fiche BANATIC, seule source à jour.
			const info = document.createDocumentFragment();
			const note = document.createElement('small');
			note.textContent = `Établissement public territorial de la Métropole du Grand Paris (SIREN : ${SIREN_MGP})`;
			info.append(normalizeText(`${ept.nom} – (SIREN : ${ept.siren})`),
				document.createElement('br'), note);
			setNodeIfCurrent(fetchId, 'epciInfo', info);

			const lien = urlFicheBanatic(ept.siren);
			['nomdupresident', 'adresseEpci', 'telephoneEpci', 'courrielEpci', 'siteEpci']
				.forEach(id => setLienIfCurrent(fetchId, id, lien,
					`voir la fiche BANATIC de ${ept.nom}`, true));
		} else if (codeEpci === SIREN_MGP) {
			// Paris n'appartient à aucun EPT : c'est la Ville de Paris qui exerce ces rôles.
			// Autre commune : liste des EPT inaccessible, ou commune absente de cette liste.
			const msg = codeCommune === CODE_PARIS
				? "sans objet (Ville de Paris)"
				: "Information non disponible";
			setTextIfCurrent(fetchId, 'epciInfo', codeCommune === CODE_PARIS
				? "Métropole du Grand Paris"
				: "Métropole du Grand Paris – établissement public territorial : information non disponible");
			['nomdupresident','adresseEpci','telephoneEpci','courrielEpci','siteEpci']
				.forEach(id => setTextIfCurrent(fetchId, id, msg));
		} else if (/^\d{9}$/.test(codeEpci)) {
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
			// Téléchargements parallèles ET mis en cache (TTL) via fetchCsvData
			const [inseeRows, uuRows] = await Promise.all([
				fetchCsvData('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/insee', ','),
				fetchCsvData('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/uu', ',')
			]);

			// fetchCsvData renvoie null en cas d'échec (au lieu de jeter)
			if (!inseeRows || !uuRows) {
				setTextIfCurrent(fetchId, 'popUrbaineInfo', "Information non disponible");
				return;
			}

			const inseeLine = inseeRows.find(r => normalizeCode(r[0]) === normalizeCode(codeCommune));
			// Ligne absente, ou 2e champ manquant (CSV tronqué)
			const codeUU = inseeLine && inseeLine[1];
			if (!codeUU) {
				setTextIfCurrent(fetchId, 'popUrbaineInfo', "Information non disponible");
				return;
			}
			const numUniteUrbaine = codeUU.substring(0, 5);
			const uuRow = uuRows.find(r => normalizeCode(r[0]) === normalizeCode(numUniteUrbaine));
			const numAssocie = uuRow ? parseInt(uuRow[1], 10) : NaN;

			if (Number.isNaN(numAssocie)) {
				setTextIfCurrent(fetchId, 'popUrbaineInfo', "Information non disponible");
				return;
			}

			// numAssocie est la tranche d'unité urbaine (TUU) de l'INSEE :
			// un entier croissant avec la taille de l'agglomération.
			const TUU_HORS_UU  = 0;  // 0 = commune hors unité urbaine (code UU « dd000 »)
			const TUU_MIN_100K = 6;  // 6 = premier palier ≥ 100 000 hab.
			const TUU_PARIS    = 8;  // 8 = agglomération de Paris

			const message =
				numAssocie === TUU_HORS_UU ? "hors unité urbaine" :
				numAssocie < TUU_MIN_100K  ? "inférieure à 100 000 habitants" :
				numAssocie < TUU_PARIS     ? "supérieure à 100 000 habitants" :
				                             "unité urbaine de Paris";  // ≥ 8

			setTextIfCurrent(fetchId, 'popUrbaineInfo', message);
		} catch (error) {
			console.error("Erreur unité urbaine :", error);
			setTextIfCurrent(fetchId, 'popUrbaineInfo', "Information non disponible");
		}
	}

	async function handleSearch() {
		if (rechercherBtn.disabled) return;
		debouncedFetchCommunes.cancel();
		infosElement.textContent = '';

		// Cas 1 : une commune a déjà été choisie dans la liste
		if (selectedCodeCommune) {
			await lancerRecherche(selectedCodeCommune);
			return;
		}

		// Cas 2 : rien de sélectionné → on tente de résoudre le texte saisi
		const saisie = communeInput.value.trim();
		if (!validateInput(saisie)) {
			showError(saisie.length === 0
				? "Veuillez entrer le nom d'une commune ou un code postal."
				: "Veuillez saisir un nom de commune ou un code postal valide.");
			return;
		}
		if (RE_CP_PARTIEL.test(saisie)) {
			showError("Le code postal doit comporter 5 chiffres.");
			return;
		}

		// Une autocomplétion encore en vol deviendrait obsolète : on l'annule.
		if (communeController) communeController.abort();
		showLoading();
		const fetchId = ++latestFetchId;
		try {
			let data;
			// Réutilise le résultat d'autocomplétion si la saisie n'a pas changé,
			// pour éviter un 2e appel identique à geo.api.gouv.fr.
			if (lastCommuneResult && lastCommuneResult.query === saisie) {
				data = lastCommuneResult.data;
			} else {
				const response = await fetchWithTimeout(buildCommuneSearchUrl(saisie));
				if (fetchId !== latestFetchId) return;   // saisie modifiée pendant la résolution → on abandonne
				if (!response.ok) throw new Error(`Erreur réseau : ${response.status}`);
				const raw = await response.json();
				if (fetchId !== latestFetchId) return;   // idem après lecture du corps
				if (!Array.isArray(raw)) throw new Error("Réponse de l'API invalide.");
				data = raw.filter(communeValide);
				lastCommuneResult = { query: saisie, data };
			}

			let exacts;
			if (RE_CP.test(saisie)) {
				// Code postal : sélection directe s'il ne désigne qu'une seule commune
				exacts = data.length === 1 ? data : [];
			} else {
				// correspondances EXACTES de nom, accents et casse ignorés
				const norm = s => String(s).toLowerCase().normalize('NFD')
					.replace(/[̀-ͯ]/g, '')
					.replace(/œ/g, 'oe').replace(/æ/g, 'ae')
					.replace(/[\s'‘’-]+/g, '');
				exacts = data.filter(c => norm(c.nom) === norm(saisie));
			}

			if (exacts.length === 1) {
				selectionnerCommune(exacts[0]);            // remplit l'en-tête, vide les cellules
				await fetchData(exacts[0].code, fetchId);  // réutilise le MÊME id ; son finally appelle hideLoading()
			} else {
				// 0 exact → suggestions complètes ; ≥ 2 homonymes → seulement les exacts
				hideLoading();
				renderCommuneList(exacts.length > 1 ? exacts : data);
			}
		} catch (error) {
			if (fetchId !== latestFetchId) return;   // erreur sur une recherche déjà invalidée → on ignore
			console.error("Erreur lors de la résolution de la commune :", error);
			hideLoading();
			showError(error && error.name === 'TimeoutError'
				? "Le service de recherche des communes ne répond pas. Veuillez réessayer."
				: undefined);
		}
	}

	function showError(userMessage = "Une erreur s'est produite. Veuillez réessayer plus tard.") {
		const enPanne = servicesEnPanne();
		infosElement.textContent = enPanne.length
			? `${userMessage} Service indisponible : ${enPanne.join(', ')} (voir « État des services » en bas de page).`
			: userMessage;
	}

	function hideCommuneList() {
		communeList.innerHTML = '';
		communeList.style.display = 'none';
		communeInput.setAttribute('aria-expanded', 'false');
		communeInput.setAttribute('aria-activedescendant', '');
		activeIndex = -1;
		communeStatus.textContent = '';
	}

	function showCommuneList() {
		communeList.style.display = 'block';
		communeInput.setAttribute('aria-expanded', 'true');
	}

	function debounce(func, delay) {
		let debounceTimer;
		function debounced() {
			const context = this, args = arguments;
			clearTimeout(debounceTimer);
			debounceTimer = setTimeout(() => func.apply(context, args), delay);
		}
		debounced.cancel = function () { clearTimeout(debounceTimer); };
		return debounced;
	}

	const debouncedFetchCommunes = debounce(function(communeName) {
		const trimmed = communeName.trim();
		if (trimmed.length === 0) {            // champ vidé → masquer sans erreur
			if (communeController) communeController.abort();
			hideCommuneList();
			return;
		}
		if (!validateInput(trimmed)) {         // caractère interdit → message ciblé
			showError("Veuillez saisir un nom de commune ou un code postal valide.");
			hideCommuneList();
			return;
		}
		if (RE_CP_PARTIEL.test(trimmed)) {     // code postal en cours de saisie → on attend les 5 chiffres
			if (communeController) communeController.abort();
			hideCommuneList();
			return;
		}
		fetchCommunes(trimmed);
	}, 300);

	communeInput.addEventListener("input", function() {
		selectedCodeCommune = null;
		activeIndex = -1;
		communeInput.setAttribute('aria-activedescendant', '');
		latestFetchId++;   // invalide tout fetch en vol → bloque ré-affichage ET écriture des cellules
		hideLoading();     // un fetch abandonné ne passera plus dans son finally → on déverrouille le bouton ici
		resetAffichage();
		effacerUrl();
		debouncedFetchCommunes(this.value);
	});

	communeInput.addEventListener('keydown', function(e) {
		if (e.isComposing) return;   // saisie IME en cours : Entrée valide la composition, pas la recherche
		const items = communeList.querySelectorAll('li[role="option"]:not([aria-disabled="true"])');

		// Enter traité AVANT le guard sur items.length : doit fonctionner même
		// sans liste ouverte (saisie directe + Entrée = recherche).
		if (e.key === 'Enter') {
			e.preventDefault();
			if (activeIndex >= 0 && items[activeIndex]) {
				items[activeIndex].click();   // item surligné → sélection + recherche (identique au clic souris)
			} else {
				handleSearch();               // aucune sélection → recherche directe (identique au bouton)
			}
			return;
		}

		if (!items.length) {
			// Liste fermée (Échap, clic extérieur) : ArrowDown la rouvre si la
			// saisie correspond encore au dernier résultat d'autocomplétion.
			if (e.key === 'ArrowDown' && !rechercherBtn.disabled && lastCommuneResult
				&& lastCommuneResult.query === communeInput.value.trim()) {
				e.preventDefault();
				renderCommuneList(lastCommuneResult.data);
				const opts = communeList.querySelectorAll('li[role="option"]:not([aria-disabled="true"])');
				if (opts.length) { activeIndex = 0; updateFocus(opts); }
			} else if (e.key === 'Escape') {
				hideCommuneList();   // ferme aussi la liste « Aucune commune trouvée »
			}
			return;
		}

		if (e.key === 'ArrowDown') {
			e.preventDefault();
			activeIndex = (activeIndex + 1) % items.length;
			updateFocus(items);
		} else if (e.key === 'ArrowUp') {
			e.preventDefault();
			activeIndex = (activeIndex - 1 + items.length) % items.length;
			updateFocus(items);
		} else if (e.key === 'Escape') {
			hideCommuneList();
		}
	});

	// --- Fermeture du menu : clic extérieur + sortie de focus ---

	// (a) Au clic sur une option, empêche le vol de focus : l'input reste
	//     focalisé, donc (c) ne se déclenche pas pendant la sélection.
	//     Élimine la course blur → liste vidée → 'click' perdu.
	communeList.addEventListener('mousedown', e => {
		if (e.target.closest('li')) e.preventDefault();
	});

	// (b) Clic/tap hors du combobox (souris, tactile, stylet) → fermeture.
	document.addEventListener('pointerdown', function (e) {
		if (communeList.style.display !== 'block') return;
		if (!comboboxEl.contains(e.target)) hideCommuneList();
	});

	// (c) Le focus quitte le combobox (ex. Tab vers un autre champ) → fermeture.
	//     relatedTarget = élément recevant le focus ; contains(null) === false,
	//     donc un focus « vers le vide » referme aussi.
	comboboxEl.addEventListener('focusout', function (e) {
		if (!comboboxEl.contains(e.relatedTarget)) hideCommuneList();
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

	async function fetchCommunes(query) {
		const myFetchId = latestFetchId;
		try {
			if (communeController) {
				communeController.abort();
			}
			communeController = new AbortController();
			const response = await fetchWithTimeout(
				buildCommuneSearchUrl(query),
				{ signal: communeController.signal });
			if (myFetchId !== latestFetchId) return;

			if (!response.ok) {
				throw new Error("Erreur réseau lors de la récupération des communes.");
			}
			const raw = await response.json();
			if (myFetchId !== latestFetchId) return;

			if (!Array.isArray(raw)) {
				throw new Error("Les données retournées par l'API sont invalides.");
			}
			const data = raw.filter(communeValide);
			lastCommuneResult = { query, data };
			renderCommuneList(data);
		} catch (error) {
			if (error.name === 'AbortError') return;      // remplacée par une saisie plus récente
			if (myFetchId !== latestFetchId) return;      // résultat obsolète
			// Un délai dépassé n'est plus ignoré silencieusement : l'utilisateur doit le savoir.
			showError(error.name === 'TimeoutError'
				? "Le service de recherche des communes ne répond pas. Veuillez réessayer."
				: undefined);
			console.error("Détails de l'erreur :", error);
		}
	}

	rechercherBtn.addEventListener("click", handleSearch);

	// Validation centralisée : nom de commune, ou chiffres (code postal, 5 max)
	function validateInput(text, maxLength = 100) {
		if (typeof text !== 'string') return false;
		const t = text.normalize('NFC').trim();   // "é" décomposé (e + ◌́) → 1 seul code point
		if (t.length < 1 || t.length > maxLength) return false;
		return RE_NOM.test(t) || /^\d{1,5}$/.test(t);
	}

	function normalizeText(text) {
		return String(text)
			.normalize('NFC')         // normalisation Unicode — accents français
			                          // pouvant arriver sous deux formes distinctes
			.replace(/ /g, ' ') // espaces insécables → espaces normaux
			                          // fréquents dans les retours d'API
			.replace(/\s+/g, ' ')    // espaces multiples → espace unique
			                          // utile après concaténation des champs d'adresse
			.trim();
	}

	/** N'accepte que des URL https vers les hôtes data.gouv.fr prévus (défense en profondeur, cf. CSP). */
	function urlRneAutorisee(url) {
		if (typeof url !== 'string') return null;
		try {
			const u = new URL(url);
			if (u.protocol === 'https:' && HOTES_RNE.includes(u.hostname)) return u.href;
		} catch (e) { /* URL invalide */ }
		console.warn("URL de ressource RNE refusée :", url);
		return null;
	}

	async function getLatestCsvUrls() {
		if (csvUrlsCache && (Date.now() - csvUrlsCache.timestamp) < CSV_CACHE_TTL) {
			return csvUrlsCache.value;
		}
		// Récupération du même endpoint déjà en cours (deux recherches rapprochées) →
		// on réutilise la promesse au lieu d'ouvrir une 2e connexion data.gouv.fr.
		if (csvUrlsInFlight) {
			return csvUrlsInFlight;
		}
		const promise = (async () => {
			try {
				const response = await fetchWithTimeout("https://www.data.gouv.fr/api/1/datasets/repertoire-national-des-elus-1/");
				if (!response.ok) throw new Error(`Erreur réseau : ${response.status}`);
				const data = await response.json();
				if (!data || !Array.isArray(data.resources))
					throw new Error("Format de réponse inattendu : resources absent ou invalide.");
				const trouver = motif => data.resources.find(r =>
					r && typeof r.title === 'string' && r.title.includes(motif));
				const resMaire = trouver("maires");
				const resPresident = trouver("conseillers-communautaires");
				const value = {
					urlMaire:     urlRneAutorisee(resMaire && resMaire.url),
					urlPresident: urlRneAutorisee(resPresident && resPresident.url),
					dateMaire:    resMaire && typeof resMaire.last_modified === 'string' ? resMaire.last_modified : null
				};
				if (value.urlMaire && value.urlPresident) {
					csvUrlsCache = { value, timestamp: Date.now() };  // on ne fige jamais un échec total
				}
				return value;
			} catch (error) {
				console.error("Erreur récupération URLs CSV :", error);
				return { urlMaire: null, urlPresident: null, dateMaire: null };
			} finally {
				csvUrlsInFlight = null;   // libère le verrou, succès comme échec
			}
		})();
		csvUrlsInFlight = promise;
		return promise;
	}

	/** Date de mise à jour du RNE : d'abord dans l'URL, sinon via last_modified. */
	function dateMiseAJourRne(urlMaire, dateMaire) {
		const m = urlMaire && urlMaire.match(/\/(\d{4})(\d{2})(\d{2})-\d{6}\//);
		if (m) return `${m[3]}/${m[2]}/${m[1]}`;
		if (dateMaire) {
			const d = new Date(dateMaire);
			if (!Number.isNaN(d.getTime())) return d.toLocaleDateString('fr-FR');
		}
		return null;
	}

	/**
	 * Les URLs sont récupérées dynamiquement via getLatestCsvUrls().
	 * Format des URLs statiques (pour référence) :
	 *   maires      : https://static.data.gouv.fr/resources/repertoire-national-des-elus-1/{date}/elus-maires-mai.csv
	 *   présidents  : https://static.data.gouv.fr/resources/repertoire-national-des-elus-1/{date}/elus-conseillers-communautaires-epci.csv
	 */
	async function fetchNomEluOuPresident(typeElu, code, csvUrl, fetchId) {
		const infoId = typeElu === "maire" ? "nomdumaire" : "nomdupresident";
		const rows = await fetchCsvData(csvUrl);

		if (!rows || rows.length < 2) {
			setTextIfCurrent(fetchId, infoId, "Information non disponible");
			return;
		}

		const header = rows[0];
		const data = rows.slice(1);

		// normalise un intitulé : retire accents, apostrophes, °, espaces, BOM…
		const norm = s => String(s == null ? '' : s)
			.normalize('NFD').replace(/[̀-ͯ]/g, '')
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
			setTextIfCurrent(fetchId, infoId, "Information non disponible");
			return;
		}

		const codeRecherche = normalizeCode(code);

		for (const row of data) {
			if (normalizeCode(row[idx.code]) !== codeRecherche) continue;
			if (typeElu !== "maire" && !norm(row[idx.fonction]).startsWith("president")) continue;

			const prenom = row[idx.prenom] || '';
			const nom    = row[idx.nom]    || '';
			if (!nom) continue;   // ligne tronquée → on continue de chercher
			const sexeElu = row[idx.sexe] === "M" ? "M."
				: row[idx.sexe] === "F" ? "Mme"
				: "";
			setTextIfCurrent(fetchId, infoId, `${sexeElu} ${prenom} ${nom}`);
			return;
		}

		console.warn("Aucun élu correspondant trouvé pour le code :", code);
		setTextIfCurrent(fetchId, infoId, "Information non disponible");
	}

	/** Site web : complète « www.exemple.fr » en https://, refuse tout autre schéma (javascript:, data:…). */
	function urlSiteSure(valeur) {
		let s = String(valeur).trim();
		if (!/^https?:\/\//i.test(s)) {
			if (!/^[a-z0-9.-]+\.[a-z]{2,}(?:[\/?#]\S*)?$/i.test(s)) return null;
			s = 'https://' + s;
		}
		try {
			const u = new URL(s);
			return (u.protocol === 'https:' || u.protocol === 'http:') ? u.href : null;
		} catch (e) {
			return null;
		}
	}

	/** Premier courriel syntaxiquement valide d'un champ (peut en contenir plusieurs). */
	function courrielValide(valeur) {
		if (typeof valeur !== 'string') return null;
		return valeur.split(/[;,\s]+/)
			.find(m => /^[^\s@<>()"',;:]+@[^\s@<>()"',;:]+\.[a-z]{2,}$/i.test(m)) || null;
	}

	async function fetchAdresse(code, type, fetchId) {
		const isMairie = type === 'mairie';
		const ids = isMairie
			? { adresse: "adressemairie", telephone: "telephonemairie", courriel: "courrielmairie", site: "sitemairie" }
			: { adresse: "adresseEpci",   telephone: "telephoneEpci",   courriel: "courrielEpci",   site: "siteEpci"   };

		try {
			// Validation AVANT construction de l'URL : un code invalide ne doit jamais
			// être interpolé dans le whereClause (défense en profondeur).
			if (!(isMairie ? RE_INSEE : /^\d{9}$/).test(String(code))) {
				throw new Error(`Code invalide : ${code}`);
			}

			const whereClause = isMairie
				? `pivot LIKE '%mairie%' AND pivot LIKE '%${code}%'`
				: `siren:"${code}"`;

			const params = new URLSearchParams({
				select: 'pivot,site_internet,nom,adresse_courriel,adresse,telephone',
				where: whereClause,
				limit: '100'
			});

			const apiUrl = `https://api-lannuaire.service-public.fr/api/explore/v2.1/catalog/datasets/api-lannuaire-administration/records?${params}`;

			const response = await fetchWithTimeout(apiUrl, { method: 'GET' });
			if (!response.ok) {
				throw new Error(`Erreur réseau : ${response.status} ${response.statusText}`);
			}
			const data = await response.json();

			if (!data || !Array.isArray(data.results) || data.results.length === 0) {
				throw new Error("Données d'adresse non disponibles ou format inattendu.");
			}

			const records = data.results.filter(record => {
				const pivotData = champJson(record.pivot);
				return isMairie
					? pivotData.some(item =>
						item && item.type_service_local === "mairie" &&
						Array.isArray(item.code_insee_commune) &&
						item.code_insee_commune.includes(code))
					: pivotData.some(item => item && item.type_service_local === "epci");
			});
			// Structure intercommunale (EPCI ou EPT) dont la fiche ne porte pas le type
			// « epci » : la requête filtrant déjà sur le SIREN exact, on garde le résultat.
			if (!isMairie && records.length === 0) {
				records.push(...data.results.filter(r => r && typeof r === 'object'));
			}

			const record = records.find(r => typeof r.nom === 'string' && r.nom.startsWith("Mairie - ")) || records[0];

			if (!record) {
				throw new Error("Aucune information sur la Mairie ou l'EPCI trouvée.");
			}

			// Les champs sont indépendants : l'absence d'adresse ne doit pas
			// masquer le courriel ni le site (fréquent pour certains EPCI).

			// Adresse : on privilégie l'adresse physique à l'adresse postale
			const adresses = champJson(record.adresse).filter(a => a && typeof a === 'object');
			const adr = adresses.find(a => a.type_adresse === 'Adresse') || adresses[0];
			let adresseComplete = '';
			if (adr) {
				const voie  = [adr.numero_voie, adr.complement1, adr.complement2, adr.service_distribution]
					.filter(v => typeof v === 'string' && v.trim()).join(', ');
				const ville = [adr.code_postal, adr.nom_commune]
					.filter(v => typeof v === 'string' && v.trim()).join(' ');
				adresseComplete = [voie, ville].filter(Boolean).join(' – ');
			}
			setTextIfCurrent(fetchId, ids.adresse, adresseComplete || "Information non disponible");

			// Téléphone
			const tel = champJson(record.telephone)
				.map(t => t && typeof t.valeur === 'string' ? t.valeur.trim() : '')
				.find(Boolean);
			if (tel) {
				const telHref = tel.replace(/[^\d+]/g, '');
				setLienIfCurrent(fetchId, ids.telephone, telHref.length >= 4 ? `tel:${telHref}` : null, tel, false);
			} else {
				setTextIfCurrent(fetchId, ids.telephone, "Information non disponible");
			}

			// Courriel
			const courriel = courrielValide(record.adresse_courriel);
			if (courriel) {
				setLienIfCurrent(fetchId, ids.courriel, `mailto:${courriel}`, courriel, false);
			} else {
				setTextIfCurrent(fetchId, ids.courriel,
					(typeof record.adresse_courriel === 'string' && record.adresse_courriel.trim()
						? record.adresse_courriel : "Information non disponible"));
			}

			// Site
			const sites = champJson(record.site_internet);
			const siteInternet = sites.length && sites[0] && typeof sites[0].valeur === 'string'
				? sites[0].valeur.trim() : '';
			if (siteInternet) {
				setLienIfCurrent(fetchId, ids.site, urlSiteSure(siteInternet), siteInternet, true);
			} else {
				setTextIfCurrent(fetchId, ids.site, "Information non disponible");
			}

		} catch (error) {
			console.error("Erreur lors de la récupération des données :", error);
			Object.values(ids).forEach(id => setTextIfCurrent(fetchId, id, "Information non disponible"));
		}
	}

	// --- Lien partageable : #insee=XXXXX ---
	// Fragment (#) plutôt que paramètre (?) : il n'est jamais transmis au serveur
	// (vie privée), et replaceState n'encombre pas l'historique.
	function majUrl(code) {
		try { history.replaceState(null, '', `#insee=${code}`); } catch (e) { /* file:// etc. */ }
	}

	function effacerUrl() {
		if (!location.hash) return;
		try { history.replaceState(null, '', location.pathname + location.search); } catch (e) { /* idem */ }
	}

	function chargerDepuisUrl() {
		const m = location.hash.match(/^#insee=([0-9AB]{5})$/i);
		if (!m) return;
		const code = m[1].toUpperCase();
		if (!RE_INSEE.test(code)) return;
		selectedCodeCommune = code;
		hideCommuneList();
		lancerRecherche(code);
	}

	async function fetchData(codeInsee, fetchId) {
		try {
			// Le code est validé avant toute interpolation dans l'URL.
			if (!RE_INSEE.test(String(codeInsee))) {
				throw new Error(`Code INSEE invalide : ${codeInsee}`);
			}
			const apiUrl = `https://geo.api.gouv.fr/communes?code=${encodeURIComponent(codeInsee)}`
				+ `&fields=nom,code,codeDepartement,population,codeEpci,epci`;

			const response = await fetchWithTimeout(apiUrl);
			if (!response.ok) {
				throw new Error(`Erreur réseau : ${response.status} ${response.statusText}`);
			}
			const data = await response.json();
			if (fetchId !== latestFetchId) return;

			if (!Array.isArray(data) || data.length === 0 || !communeValide(data[0])) {
				showError("Commune introuvable.");
				return;
			}

			const commune = data[0];
			const codeCommune = commune.code;
			// En-tête et champ de saisie alignés sur la réponse (utile pour un lien partagé)
			selectedCodeCommune = codeCommune;
			communeInput.value = normalizeText(commune.nom);
			afficherEnTete(commune);
			majUrl(codeCommune);

			const codeEpci = commune.codeEpci;
			const hasEpci = !!(commune.epci && codeEpci);
			if (!hasEpci) afficherCommuneSansEpci(fetchId);

			// Commune de la MGP (hors Paris) : EPT de rattachement, chargé en parallèle.
			const estCommuneEpt = codeEpci === SIREN_MGP && codeCommune !== CODE_PARIS;
			const [{ urlMaire: csvUrlMaire, urlPresident: csvUrlPresident, dateMaire }, ept] = await Promise.all([
				getLatestCsvUrls(),
				estCommuneEpt ? eptDeCommune(codeCommune) : null
			]);
			if (fetchId !== latestFetchId) return;

			const dateMaj = dateMiseAJourRne(csvUrlMaire, dateMaire);
			if (dateMaj) {
				setTextIfCurrent(fetchId, 'sourceRNEDate', `(mise à jour du ${dateMaj})`);
			}

			// Commune de la MGP : les compétences PLU et RLP sont exercées par l'EPT.
			const codeCompetence = ept && !ept.indisponible ? ept.siren : codeEpci;

			handlePopulationData(data, fetchId);
			await Promise.all([
				hasEpci ? handleEpciData(data, csvUrlPresident, fetchId, codeCommune, ept) : null,
				handleMaireData(codeCommune, csvUrlMaire, fetchId),
				handleUniteUrbaineData(codeCommune, fetchId),
				hasEpci ? handleCompetenceData(codeCompetence, 'PLU', fetchId, codeCommune) : null,
				hasEpci ? handleCompetenceData(codeCompetence, 'RLP', fetchId, codeCommune) : null
			]);

			if (fetchId !== latestFetchId) return;
			document.querySelectorAll("table.resultat").forEach(table => {
				table.style.display = "table";
			});
		} catch (error) {
			console.error("Une erreur s'est produite lors de la récupération des données de l'API :", error);
			if (fetchId === latestFetchId) {
				showError(error && error.name === 'TimeoutError'
					? "Un service ne répond pas. Veuillez réessayer."
					: undefined);
			}
		} finally {
			if (fetchId === latestFetchId) hideLoading();
		}
	}

	// --- État des services ---
	// Alimenté passivement par chaque appel de fetchWithTimeout (recherches réelles)
	// et activement par verifierServices() (petites requêtes de test).
	const SERVICES = {
		'geo.api.gouv.fr':                 { cle: 'geo',      nom: 'Découpage administratif' },
		'www.data.gouv.fr':                { cle: 'datagouv', nom: 'Catalogue data.gouv.fr' },
		'static.data.gouv.fr':             { cle: 'rne',      nom: 'Fichiers du RNE' },
		'api-lannuaire.service-public.fr': { cle: 'annuaire', nom: "Annuaire de l'administration" },
		'raw.githubusercontent.com':       { cle: 'github',   nom: 'Fichiers GitHub' },
		'cdn.jsdelivr.net':                { cle: 'etalab',   nom: 'Liste des EPT (Etalab)' }
	};
	const SEUIL_LENT_MS = 3000;
	const etatParCle = {};   // cle -> 'ok' | 'lent' | 'ko' | 'attente' | 'inconnu'
	const verifierServicesBtn = document.getElementById('verifierServicesBtn');
	const etatResume = document.getElementById('etatResume');

	function serviceDeUrl(url) {
		try { return SERVICES[new URL(url).hostname] || null; } catch (e) { return null; }
	}

	function formaterDuree(ms) {
		return ms < 1000
			? `${Math.round(ms)} ms`
			: `${(ms / 1000).toLocaleString('fr-FR', { maximumFractionDigits: 1 })} s`;
	}

	function afficherEtat(cle, etat, texte) {
		etatParCle[cle] = etat;
		const cell = document.getElementById(`etat-${cle}`);
		const heure = document.getElementById(`heure-${cle}`);
		if (!cell) return;
		// Le symbole double la couleur : l'information n'est pas portée par la seule couleur.
		const symbole = { ok: '✅', lent: '⚠️', ko: '❌', attente: '⏳', inconnu: '❔' }[etat] || '';
		cell.textContent = `${symbole} ${texte}`;
		cell.className = `etat-${etat}`;
		if (heure && etat !== 'attente') {
			heure.textContent = new Date().toLocaleTimeString('fr-FR');
		}
		majResumeEtat();
	}

	function enregistrerEtat(url, { status, ms, erreur }) {
		const service = serviceDeUrl(url);
		if (!service) return;
		if (erreur) {
			afficherEtat(service.cle, 'ko', erreur.name === 'TimeoutError'
				? 'indisponible (délai dépassé)'
				: 'indisponible (erreur réseau)');
		} else if (status >= 500) {
			afficherEtat(service.cle, 'ko', `indisponible (erreur HTTP ${status})`);
		} else if (status === 429) {
			afficherEtat(service.cle, 'lent', 'saturé (trop de requêtes)');
		} else if (ms > SEUIL_LENT_MS) {
			afficherEtat(service.cle, 'lent', `lent (${formaterDuree(ms)})`);
		} else {
			// Un 404 sur une ressource précise prouve que le service répond.
			afficherEtat(service.cle, 'ok', `opérationnel (${formaterDuree(ms)})`);
		}
	}

	function servicesEnPanne() {
		return Object.values(SERVICES)
			.filter(s => etatParCle[s.cle] === 'ko')
			.map(s => s.nom);
	}

	function majResumeEtat() {
		const etats = Object.values(SERVICES).map(s => etatParCle[s.cle]);
		if (etats.some(e => e === 'attente')) {
			etatResume.textContent = '⏳ Vérification des services en cours…';
			return;
		}
		if (etats.every(e => e === undefined)) {
			etatResume.textContent = '';
			return;
		}
		const ko = servicesEnPanne();
		const lents = Object.values(SERVICES).filter(s => etatParCle[s.cle] === 'lent').map(s => s.nom);
		if (ko.length) {
			etatResume.textContent = `❌ ${ko.length} service${ko.length > 1 ? 's' : ''} indisponible${ko.length > 1 ? 's' : ''} : ${ko.join(', ')}. Certaines informations peuvent manquer.`;
		} else if (lents.length) {
			etatResume.textContent = `⚠️ Service${lents.length > 1 ? 's' : ''} ralenti${lents.length > 1 ? 's' : ''} : ${lents.join(', ')}.`;
		} else if (etats.every(e => e === 'ok')) {
			etatResume.textContent = '✅ Tous les services sont opérationnels.';
		} else {
			etatResume.textContent = 'Aucun service en panne détecté.';
		}
	}

	/** Test actif : une petite requête par service (HEAD quand c'est possible). */
	async function verifierServices() {
		verifierServicesBtn.disabled = true;
		Object.values(SERVICES).forEach(s => afficherEtat(s.cle, 'attente', 'vérification…'));
		const tester = (url, options) =>
			fetchWithTimeout(url, options, 8000).catch(() => { /* état déjà enregistré */ });
		try {
			await Promise.all([
				tester('https://geo.api.gouv.fr/communes?code=75056&fields=code'),
				tester('https://api-lannuaire.service-public.fr/api/explore/v2.1/catalog/datasets/api-lannuaire-administration/records?select=nom&limit=1'),
				tester('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/uu', { method: 'HEAD' }),
				tester(URL_EPT, { method: 'HEAD' }),
				// Le catalogue fournit l'URL du fichier des maires, testé ensuite.
				tester('https://www.data.gouv.fr/api/1/datasets/repertoire-national-des-elus-1/', { method: 'HEAD' })
					.then(() => getLatestCsvUrls())
					.then(({ urlMaire }) => urlMaire
						? tester(urlMaire, { method: 'HEAD' })
						: afficherEtat('rne', 'inconnu', 'non vérifiable (adresse du fichier inconnue)'))
			]);
		} finally {
			// Toute ligne restée « en attente » (cas imprévu) est signalée comme inconnue.
			Object.values(SERVICES)
				.filter(s => etatParCle[s.cle] === 'attente')
				.forEach(s => afficherEtat(s.cle, 'inconnu', 'état inconnu'));
			verifierServicesBtn.disabled = false;
		}
	}

	verifierServicesBtn.addEventListener('click', verifierServices);

	window.addEventListener('hashchange', chargerDepuisUrl);
	chargerDepuisUrl();
	verifierServices();

	});
	</script>


	<hr> <b>Sources :</b>
	<ul style="list-style-type:square">
		<li>(1) API gouvernementale : <a href="https://geo.api.gouv.fr/decoupage-administratif/communes" target="_blank" rel="noopener noreferrer">https://geo.api.gouv.fr/decoupage-administratif/communes</a> – rattachement des communes de la Métropole du Grand Paris à leur établissement public territorial (EPT) : <a href="https://github.com/datagouv/decoupage-administratif" target="_blank" rel="noopener noreferrer">découpage administratif (Etalab)</a></li>
		<li>(2) informations mises à jour manuellement – valable au 1er janvier 2026 – source : <a href="https://www.insee.fr/fr/information/4802589" target="_blank" rel="noopener noreferrer">https://www.insee.fr/fr/information/4802589</a></li>
		<li>(3) OpenData gouvernemental : Ministère de l'Intérieur et des Outre-Mer – <a href="https://www.data.gouv.fr/fr/datasets/repertoire-national-des-elus-1/" target="_blank" rel="noopener noreferrer">https://www.data.gouv.fr/fr/datasets/repertoire-national-des-elus-1/</a> <span id="sourceRNEDate"></span></li>
		<li>(4) API gouvernementale : <a href="https://api-lannuaire.service-public.fr/explore/dataset/api-lannuaire-administration" target="_blank" rel="noopener noreferrer">https://api-lannuaire.service-public.fr/explore/dataset/api-lannuaire-administration</a></li>
		<li>(5) informations mises à jour manuellement (intercommunalité puis Export national ou régional) – valable au 4 juillet 2026 ; président et coordonnées des établissements publics territoriaux (EPT) de la Métropole du Grand Paris : lien direct vers leur fiche BANATIC (données à jour) – source : <a href="https://www.banatic.interieur.gouv.fr/export/" target="_blank" rel="noopener noreferrer">https://www.banatic.interieur.gouv.fr/export/</a></li>
	</ul>

	<hr> <b>Historique :</b>
	<ul style="list-style-type:square">
		<li>version 1.46h du 12/11/2026 : Mise à jour du code</li>
		<li>version 1.45a du 12/07/2026 : Mise à jour du code</li>
		<li>version 1.44g du 04/07/2026 : Mise à jour du code</li>
		<li>version 1.43a du 01/07/2026 : Mise à jour du code</li>
		<li>version 1.42e du 30/06/2026 : Mise à jour du code</li>
		<li>version 1.41a du 29/06/2026 : Mise à jour du code</li>
		<li>version 1.40h du 27/06/2026 : Mise à jour du code</li>
		<li>version 1.39e du 26/06/2026 : Mise à jour du code</li>
		<li>version 1.38g du 25/06/2026 : Mise à jour du code</li>
		<li>version 1.37h du 23/06/2026 : Mise à jour du code</li>
		<li>version 1.36f du 22/06/2026 : Mise à jour du code</li>
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
		}
	});
	</script>
</body>
</html>
