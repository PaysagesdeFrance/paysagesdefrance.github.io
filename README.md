<html lang="fr">

<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta http-equiv="Content-Security-Policy" content="default-src 'none'; script-src 'self' ; connect-src 'self' https://geo.api.gouv.fr https://api-lannuaire.service-public.fr; frame-ancestors 'none';">
	<meta http-equiv="X-Content-Type-Options" content="nosniff">
	<meta name="referrer" content="strict-origin">
	<meta http-equiv="Strict-Transport-Security" content="max-age=63072000; includeSubDomains; preload">
	<title>Recherche d'une commune</title>
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
	</style>
</head>

<body>
	<h1>Recherche d'une commune</h1>
		<div class="combobox">
		<input type="text" id="communeInput" name="commune" autocomplete="off">
		<ul id="commune-list" class="dropdown-menu"></ul>
	</div>
	<button id="rechercherBtn">Rechercher</button>
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
	</table>
	<br>
<script>
    document.addEventListener('DOMContentLoaded', function() {
        const communeInput = document.getElementById("communeInput");
        const communeList = document.getElementById("commune-list");
        const rechercherBtn = document.getElementById("rechercherBtn");
        const infosElement = document.getElementById("infos");
        let selectedCodeCommune;

        function updateElementText(elementId, text = 'Données non disponibles') {
            const element = document.getElementById(elementId);
            element.textContent = element && typeof text === 'string' ? escapeHTML(text) : text;
        }

        async function fetchCsvData(url) {
            try {
                const response = await fetch(url);
                if (!response.ok) throw new Error(`Erreur réseau : ${response.status} ${response.statusText}`);
                return (await response.text()).trim().split('\n').slice(1).map(line => line.split(';'));
            } catch (error) {
                console.error("Erreur lors de la récupération du fichier CSV :", error);
                showError();
                return null;
            }
        }

        async function fetchData(url, parseData) {
            try {
                const response = await fetch(url);
                if (!response.ok) throw new Error(`Erreur réseau : ${response.status} ${response.statusText}`);
                return parseData ? await parseData(await response.text()) : await response.json();
            } catch (error) {
                console.error("Erreur lors de la récupération des données :", error);
                showError();
                return null;
            }
        }

        function showError(userMessage = "Une erreur s'est produite. Veuillez réessayer plus tard.") {
            infosElement.textContent = userMessage;
            console.error("Détails de l'erreur :", new Error().stack);
        }

        function handlePopulationData(data) {
            if (Array.isArray(data) && data.length > 0 && typeof data[0].population === 'number') {
                updateElementText('populationInfo', `${data[0].population} habitants`);
            } else {
                showError();
            }
        }

        function handleEpciData(data) {
            if (Array.isArray(data) && data.length > 0 && data[0].epci) {
                const { nom, codeEpci } = data[0].epci;
                updateElementText('epciInfo', `${nom || 'Non disponible'} – (SIREN : ${codeEpci})`);
                if (codeEpci && codeEpci !== "200054781") {
                    fetchAdresse(codeEpci, "epci");
                    fetchNomEluOuPresident("president", codeEpci);
                } else {
                    updateElementText('epciInfo', `Métropole du Grand Paris – dépend d'un EPT`);
                }
            } else {
                showError();
            }
        }

        function resetCommuneInfo() {
            ['resultatCommune', 'populationInfo', 'popUrbaineInfo', 'epciInfo', 'nomdumaire', 'adressemairie', 'courrielmairie', 'sitemairie', 'nomdupresident', 'adresseEpci', 'courrielEpci', 'siteEpci', 'competencePLU']
            .forEach(id => updateElementText(id));
        }

        async function handleSearch() {
            const nomCommune = escapeHTML(communeInput.value.trim());
            if (selectedCodeCommune) {
                fetchData(`https://geo.api.gouv.fr/communes?code=${selectedCodeCommune}&fields=code,population,codeEpci,epci,siren`, async (data) => {
                    if (validateApiResponse(data, ['code', 'population', 'epci', 'siren'])) {
                        handlePopulationData(data);
                        handleEpciData(data);
                        await handleUniteUrbaineData(data[0].code);
                        if (data[0].codeEpci) {
                            await handlePluData(data[0].codeEpci);
                        }
                    } else {
                        showError();
                    }
                });
            } else {
                showError('Veuillez entrer le nom d\'une commune.');
            }
        }

        async function fetchCommunes(communeName) {
            try {
                const data = await fetchData(`https://geo.api.gouv.fr/communes?nom=${communeName}&limit=13`);
                communeList.innerHTML = '';
                data.forEach(commune => {
                    const { nom, codeDepartement, code } = commune;
                    if (typeof nom === 'string' && typeof codeDepartement === 'string' && typeof code === 'string') {
                        const listItem = document.createElement("li");
                        listItem.textContent = `${escapeHTML(nom)} (${escapeHTML(codeDepartement)})`;
                        listItem.addEventListener("click", () => {
                            selectedCodeCommune = code;
                            communeInput.value = nom;
                            hideCommuneList();
                            resetCommuneInfo();
                            const h2Element = document.createElement('h2');
                            h2Element.textContent = `– ${nom} (${codeDepartement}) – code INSEE ${selectedCodeCommune}`;
                            document.getElementById('resultatCommune').textContent = '';
                            document.getElementById('resultatCommune').appendChild(h2Element);
                            rechercherBtn.focus();
                        });
                        communeList.appendChild(listItem);
                    }
                });
                communeList.style.display = 'block';
            } catch (error) {
                showError();
            }
        }

        function debounce(func, delay) {
            let debounceTimer;
            return function() {
                clearTimeout(debounceTimer);
                debounceTimer = setTimeout(() => func.apply(this, arguments), delay);
            };
        }

        communeInput.addEventListener("input", debounce(function() {
            const communeName = this.value.trim();
            if (validateText(communeName, 50)) {
                communeName.length >= 1 ? fetchCommunes(communeName) : hideCommuneList();
            } else {
                showError();
                hideCommuneList();
            }
        }, 300));

        document.addEventListener("click", function(event) {
            if (!communeInput.contains(event.target) && !communeList.contains(event.target)) {
                communeList.style.display = 'none';
            }
        });

        rechercherBtn.addEventListener("click", handleSearch);

        function validateText(text, maxLength = 100) {
            const regex = /^[A-Za-zÀ-ÖØ-öø-ÿ0-9\s'-]+$/;
            const forbiddenPatterns = /(<\/?script.*?>|javascript:|onerror\s*=|onload\s*=)/i;
            return regex.test(text) && text.length > 0 && text.length <= maxLength && !forbiddenPatterns.test(text);
        }

        function escapeHTML(str) {
            return str.replace(/[&<>"'`/\\(){}]/g, match => ({
                '&': '&amp;',
                '<': '&lt;',
                '>': '&gt;',
                '"': '&quot;',
                "'": '&#39;',
                '`': '&#96;',
                '/': '&#x2F;',
                '\\': '&#92;',
                '(': '&#40;',
                ')': '&#41;',
                '{': '&#123;',
                '}': '&#125;'
            }[match]));
        }

        async function fetchNomEluOuPresident(typeElu, code) {
            const csvUrl = typeElu === "maire" ? "https://static.data.gouv.fr/resources/repertoire-national-des-elus-1/20240730-125205/elus-maires.csv" : "https://static.data.gouv.fr/resources/repertoire-national-des-elus-1/20240731-142441/elus-epci.csv";
            const data = await fetchCsvData(csvUrl);
            if (data) {
                const row = data.find(row => parseInt(row[4]) === parseInt(code) && (typeElu === "maire" || row[15] === "Président du conseil communautaire"));
                if (row) {
                    const nomElu = row[typeElu === "maire" ? 6 : 8];
                    const prenomElu = row[typeElu === "maire" ? 7 : 9];
                    const sexeElu = row[typeElu === "maire" ? 8 : 10] === "M" ? "M." : (row[typeElu === "maire" ? 8 : 10] === "F" ? "Mme" : "");
                    if (validateText(nomElu) && validateText(prenomElu)) {
                        document.getElementById(typeElu === "maire" ? "nomdumaire" : "nomdupresident").textContent = `${sexeElu} ${escapeHTML(nomElu)} ${escapeHTML(prenomElu)}`;
                    } else {
                        showError();
                    }
                } else {
                    showError();
                }
            }
        }

        async function fetchAdresse(code, type) {
            const endpoint = type === 'mairie' ? `code_insee_commune%3A%22${code}%22` : `siren%3A%22${code}%22`;
            const apiUrl = `https://api-lannuaire.service-public.fr/api/explore/v2.1/catalog/datasets/api-lannuaire-administration/records?where=${endpoint}`;
            const data = await fetchData(apiUrl, false);
            if (data) {
                const { adresse, courriel, site } = data.records[0].fields || {};
                if (adresse) updateElementText(type === 'mairie' ? 'adressemairie' : 'adresseEpci', adresse);
                if (courriel) updateElementText(type === 'mairie' ? 'courrielmairie' : 'courrielEpci', courriel);
                if (site) updateElementText(type === 'mairie' ? 'sitemairie' : 'siteEpci', site);
            }
        }

        async function handleUniteUrbaineData(codeCommune) {
            const apiUrl = `https://api.insee.fr/ugp/rest/communes/${codeCommune}/unite-urbaine`;
            const data = await fetchData(apiUrl);
            if (data && Array.isArray(data.uniteUrbaine)) {
                updateElementText('popUrbaineInfo', `Population : ${data.uniteUrbaine[0].population}`);
            } else {
                showError();
            }
        }

        async function handlePluData(codeEpci) {
            const apiUrl = `https://api.datalocale.fr/v1/plu/epci/${codeEpci}`;
            const data = await fetchData(apiUrl);
            if (data && Array.isArray(data.plu)) {
                const plu = data.plu[0].intitule;
                updateElementText('competencePLU', plu || 'Non spécifié');
            } else {
                showError();
            }
        }

        function validateApiResponse(data, requiredFields) {
            return requiredFields.every(field => field in data[0]);
        }

        function hideCommuneList() {
            communeList.style.display = 'none';
        }
    });
</script>



	<hr> <b>Sources :</b>
	<ul style="list-style-type:square">
		<li>(1) API gouvernementale : <a href="https://geo.api.gouv.fr/decoupage-administratif/communes" target="_blank">https://geo.api.gouv.fr/decoupage-administratif/communes</a></li>
		<li>(2) informations mises à jour manuellement – valable au 1er janvier 2024 – source : <a href="https://www.insee.fr/fr/information/4802589" target="_blank">https://www.insee.fr/fr/information/4802589</a></li>
		<li>(3) OpenData gouvernemental : Ministère de l'Intérieur et des Outre-Mer – <a href="https://www.data.gouv.fr/fr/datasets/repertoire-national-des-elus-1/" target="_blank">https://www.data.gouv.fr/fr/datasets/repertoire-national-des-elus-1/</a></li>
		<li>(4) API gouvernementale : <a href="https://api-lannuaire.service-public.fr/explore/dataset/api-lannuaire-administration" target="_blank">https://api-lannuaire.service-public.fr/explore/dataset/api-lannuaire-administration</a></li>
		<li>(5) informations mises à jour manuellement – valable au 1er juillet 2024 – source : <a href="https://www.banatic.interieur.gouv.fr/" target="_blank">https://www.banatic.interieur.gouv.fr/</a></li>

	</ul>

	<hr> <b>Historique :</b>
	<ul style="list-style-type:square">
 		<li>version 1.19d du 27/10/2024 : Amélioration de la simplicité</li>
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
