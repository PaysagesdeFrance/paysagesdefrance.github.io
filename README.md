<html lang="fr">

<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta http-equiv="Content-Security-Policy" content="default-src 'none'; script-src 'self' ; connect-src 'self' https://geo.api.gouv.fr https://api-lannuaire.service-public.fr; frame-ancestors 'none';">
	<meta http-equiv="X-Content-Type-Options" content="nosniff">
	<meta name="referrer" content="strict-origin">
	<meta http-equiv="Strict-Transport-Security" content="max-age=63072000; includeSubDomains; preload">
	<title>Recherche d'une commune</title>
	<script defer src="/lib/papaparse.min.js"></script>
<script src="/lib/axios.min.js"></script>
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
		let lastSearchTimeout;
		let selectedCodeCommune;

// Sous-fonction pour gérer les données de la compétence PLU
async function handlePluData(codeEpci) {
    const pluResponse = await axios.get('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/plu');
    const lines = pluResponse.data.split('\n');
    const line = lines.find(line => line.startsWith(`${codeEpci},`));
    if (line) {
        const uuValues = line.split(',');
        const numAssocie = uuValues[1];
        let message = "";
        if (numAssocie === "0") {
            message = "non";
        } else if (numAssocie === "1") {
            message = "oui";
        } else {
            message = "Valeur inconnue";
        }
        document.getElementById('competencePLU').textContent = escapeHTML(message);
    } else {
        document.getElementById('competencePLU').textContent = "Information non disponible";
    }
}


// Sous-fonction pour gérer les données de population
function handlePopulationData(data) {
    if (!Array.isArray(data) || data.length === 0 || typeof data[0].population !== 'number') {
        showError('Les données de population sont invalides ou indisponibles.');
        document.getElementById('populationInfo').textContent = 'Données non disponibles';
        return;
    }
    
    const population = data[0].population;
    if (Number.isInteger(population) && population >= 0) {
        document.getElementById('populationInfo').textContent = escapeHTML(population.toString()) + ' habitants';
    } else {
        document.getElementById('populationInfo').textContent = 'Données non disponibles';
    }
}


// Sous-fonction pour gérer les données EPCI
function handleEpciData(data) {
    const epci = data[0].epci;
    const nomEpci = epci ? epci.nom : 'Non disponible';
    const codeEpci = data[0].codeEpci;

    if (nomEpci && validateText(nomEpci)) {
        document.getElementById('epciInfo').textContent = escapeHTML(nomEpci) + ' – (SIREN : ' + escapeHTML(codeEpci) + ')';
    } else {
        document.getElementById('epciInfo').textContent = 'EPCI non disponible';
    }

    // Si le code EPCI est valide, récupérer les informations EPCI et président
    if (codeEpci && codeEpci !== "200054781") {
        fetchAdresseData(codeEpci, "epci");
        fetchNomEluOuPresident("president", codeEpci);
    } else {
        document.getElementById('epciInfo').textContent = `Métropole du Grand Paris – dépend d'un EPT`;
    }
}

// Sous-fonction pour gérer les informations sur le maire
function handleMaireData(codeCommune) {
    fetchNomEluOuPresident("maire", codeCommune);
    fetchAdresseData(codeCommune, "mairie");
}

// Sous-fonction pour gérer les données de l'unité urbaine
async function handleUniteUrbaineData(codeCommune) {
    const inseeResponse = await axios.get('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/insee');
    const inseeLines = inseeResponse.data.split('\n');
    const inseeLine = inseeLines.find(line => line.startsWith(`${codeCommune},`));
    if (inseeLine) {
        const values = inseeLine.split(',');
        const numUniteUrbaine = values[1].substring(0, 5);
        const uuResponse = await axios.get('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/uu');
        const uuLines = uuResponse.data.split('\n');
        const uuLine = uuLines.find(uuLine => uuLine.includes(`${numUniteUrbaine},`));
        if (uuLine) {
            const uuValues = uuLine.split(',');
            const numAssocie = parseInt(uuValues[1], 10);
            let populationUrbainMessage = "";
            if (numAssocie <= 5) {
                populationUrbainMessage = "inférieure à 100000 habitants";
            } else if (numAssocie === 8) {
                populationUrbainMessage = "unité urbaine de Paris";
            } else if (numAssocie === 6 || numAssocie === 7) {
                populationUrbainMessage = "supérieure à 100000 habitants";
            } else {
                populationUrbainMessage = "Aucune condition spécifiée";
            }
            document.getElementById('popUrbaineInfo').textContent = escapeHTML(populationUrbainMessage);
        } else {
            document.getElementById('popUrbaineInfo').textContent = "hors unité urbaine";
        }
    } else {
        document.getElementById('popUrbaineInfo').textContent = "Information non disponible";
    }
}


function handleSearch() {
    const nomCommune = communeInput.value.trim();
    infosElement.textContent = '';
    
    if (selectedCodeCommune) {
        fetchData(selectedCodeCommune);
        document.querySelectorAll("table").forEach(table => {
            table.style.display = "table";
        });
    } else {
        showError('Veuillez entrer le nom d\'une commune.');
    }
}

function showError(message) {
    infosElement.textContent = message || "Une erreur s'est produite. Veuillez réessayer.";
    console.error("Détails de l'erreur :", message);
}



function hideCommuneList() {
    communeList.innerHTML = '';
    communeList.style.display = 'none';
}

function showCommuneList() {
    communeList.style.display = 'block';
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

communeInput.addEventListener("input", debounce(function() {
    var communeName = this.value;
    if (!validateText(communeName, 50)) { // Limite de longueur à 50 caractères
        showError("Le nom de la commune contient des caractères invalides ou est trop long.");
        hideCommuneList();
        return;
    }
    if (communeName.length >= 1) {
        fetchCommunes(communeName);
    } else {
        hideCommuneList();
    }
}, 300));


function fetchCommunes(communeName) {
    fetch(`https://geo.api.gouv.fr/communes?nom=${communeName}&limit=13`)
        .then(response => {
            if (!response.ok) {
                throw new Error("Erreur réseau lors de la récupération des communes.");
            }
            return response.json();
        })
        .then(data => {
            communeList.innerHTML = ''; // Vide la liste des résultats précédents
            data.forEach(function(commune) {
                const listItem = document.createElement("li");
                listItem.textContent = `${escapeHTML(commune.nom)} (${escapeHTML(commune.codeDepartement)})`;
                listItem.addEventListener("click", function() {
                    selectedCodeCommune = commune.code;
                    communeInput.value = commune.nom;
                    hideCommuneList(); // Masque la liste des suggestions
                    infosElement.textContent = '';
                    
                    // Réinitialise les informations affichées
                    document.getElementById('resultatCommune').textContent = '';
                    document.getElementById('populationInfo').textContent = '';
                    document.getElementById('popUrbaineInfo').textContent = '';
                    document.getElementById('epciInfo').textContent = '';
                    document.getElementById('nomdumaire').textContent = '';
                    document.getElementById('adressemairie').textContent = '';
                    document.getElementById('courrielmairie').textContent = '';
                    document.getElementById('sitemairie').textContent = '';
                    document.getElementById('nomdupresident').textContent = '';
                    document.getElementById('adresseEpci').textContent = '';
                    document.getElementById('courrielEpci').textContent = '';
                    document.getElementById('siteEpci').textContent = '';
                    document.getElementById('competencePLU').textContent = '';
                    
                    const resultatCommune = document.getElementById('resultatCommune');
                    const h2Element = document.createElement('h2');
                    h2Element.textContent = `– ${commune.nom} (${commune.codeDepartement}) – code INSEE ${selectedCodeCommune}`;
                    resultatCommune.textContent = ''; // Efface le contenu précédent
                    resultatCommune.appendChild(h2Element);
                    
                    // Met le focus sur le bouton rechercher si la recherche a abouti
                    if (resultatCommune.textContent.trim() !== "") {
                        rechercherBtn.focus();
                    }
                });
                communeList.appendChild(listItem); // Ajoute l'élément à la liste
            });
            showCommuneList(); // Affiche la liste des suggestions
        })
        .catch(error => {
            showError("Une erreur s'est produite lors de la recherche des communes. Veuillez réessayer.");
            console.error("Détails de l'erreur :", error);
        });
}

document.addEventListener("click", function(event) {
    if (!communeInput.contains(event.target) && !communeList.contains(event.target)) {
        hideCommuneList();
    }
});

rechercherBtn.addEventListener("click", handleSearch);

function validateText(text, maxLength = 100) {
    const regex = /^[A-Za-zÀ-ÖØ-öø-ÿ0-9]+(?:[ '-][A-Za-zÀ-ÖØ-öø-ÿ0-9]+)*$/;
    // Ce regex autorise des lettres, chiffres, espaces, apostrophes et tirets, 
    // mais interdit qu'ils apparaissent en début ou en fin, ou se suivent.

    // Vérifie si le texte est valide et respecte les contraintes de longueur
    return regex.test(text.trim()) && text.length > 0 && text.length <= maxLength;
}



function escapeHTML(str) {
    return str.replace(/[&<>"']/g, function(match) {
        const escapeChars = {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
            '"': '&quot;',
            "'": '&#39;'
        };
        return escapeChars[match];
    });
}


function fetchNomEluOuPresident(typeElu, code) {
    const csvUrlMaire = "https://static.data.gouv.fr/resources/repertoire-national-des-elus-1/20240730-125205/elus-maires.csv";
    const csvUrlPresident = "https://static.data.gouv.fr/resources/repertoire-national-des-elus-1/20240731-142441/elus-epci.csv";
    const csvUrl = typeElu === "maire" ? csvUrlMaire : csvUrlPresident;
    Papa.parse(csvUrl, {
        download: true,
        header: false,
        complete: function(results) {
            const data = results.data;
            for(let i = 0; i < data.length; i++) {
                const codeIndex = 4;
                const fonctionIndex = 15;
                
                if(parseInt(data[i][codeIndex]) === parseInt(code) &&
                   (typeElu === "maire" || data[i][fonctionIndex] === "Président du conseil communautaire")) {

                    const nomElu = data[i][typeElu === "maire" ? 6 : 8];
                    const prenomElu = data[i][typeElu === "maire" ? 7 : 9];
                    let sexeElu = data[i][typeElu === "maire" ? 8 : 10];

                    // Validation et échappement des données avant l'affichage
if (typeof nomElu === 'string' && typeof prenomElu === 'string' && validateText(nomElu) && validateText(prenomElu)) {
    if (sexeElu === "M") {
        sexeElu = "M.";
    } else if (sexeElu === "F") {
        sexeElu = "Mme";
    } else {
        sexeElu = ""; // Valeur par défaut en cas de sexe non valide
    }
    const infoText = typeElu === "maire" ? "nomdumaire" : "nomdupresident";
    document.getElementById(infoText).textContent = `${sexeElu} ${escapeHTML(nomElu)} ${escapeHTML(prenomElu)}`;
} else {
    console.warn("Données de l'élu invalides : ", nomElu, prenomElu);
    showError("Les informations de l'élu sont invalides.");
}
                    break; // Arrête la boucle une fois l'élu trouvé
                }
            }
        },
        error: function(error) {
            showError("Une erreur s'est produite lors de la récupération du fichier CSV. Veuillez réessayer.");
            console.error("Erreur lors de la récupération du fichier CSV :", error);
        }
    });
}


async function fetchAdresseData(code, type) {
    const isMairie = type === 'mairie';
    const endpoint = isMairie ? `code_insee_commune%3A%22${code}%22` : `siren%3A%22${code}%22`;
    const apiUrl = `https://api-lannuaire.service-public.fr/api/explore/v2.1/catalog/datasets/api-lannuaire-administration/records?select=pivot%2Csite_internet%2Cnom%2Cadresse_courriel%2Cadresse&where=${endpoint}&limit=100`;

    try {
        const response = await fetch(apiUrl);
        const data = await response.json();

        const mairieRecord = data.results.find(record => {
            const pivotData = record.pivot ? JSON.parse(record.pivot) : [];
            return (
                (isMairie && pivotData.some(item => item.type_service_local === "mairie") && record.nom.startsWith("Mairie - ")) || 
                (!isMairie && pivotData.some(item => item.type_service_local === "epci"))
            );
        });

        if (mairieRecord && mairieRecord.adresse) {
            const adresseData = JSON.parse(mairieRecord.adresse);
            // Construire l'adresse avec des tirets entre les différents éléments
            const adresseMairie = [
                adresseData[0].numero_voie || '',
                adresseData[0].complement1 || '',
                adresseData[0].complement2 || '',
                adresseData[0].service_distribution || '',
                adresseData[0].code_postal || '',
                adresseData[0].nom_commune || ''
            ].filter(Boolean).join(' - '); // Ajoute un tiret entre les champs non vides

            // Affichage de l'adresse, validation souple
            if (adresseMairie) {
                const infoText = type === "mairie" ? "adressemairie" : "adresseEpci";
                document.getElementById(infoText).textContent = escapeHTML(adresseMairie);
            } else {
                console.warn("Adresse vide ou non valide :", adresseMairie);
            }

            if (mairieRecord.adresse_courriel) {
                const infoText = type === "mairie" ? "courrielmairie" : "courrielEpci";
                document.getElementById(infoText).textContent = escapeHTML(mairieRecord.adresse_courriel);
            }

            const siteInternetJSON = mairieRecord.site_internet;
            if (siteInternetJSON) {
                const siteInternetData = JSON.parse(siteInternetJSON);
                const siteInternet = siteInternetData.length > 0 ? siteInternetData[0].valeur : '';
                const infoText = type === "mairie" ? "sitemairie" : "siteEpci";
                if (siteInternet) {
                    document.getElementById(infoText).innerHTML = `<a href="${escapeHTML(siteInternet)}" target="_blank">${escapeHTML(siteInternet)}</a>`;
                }
            }
        } else {
            if (isMairie) {
                fetchAdresseCommune(sirenCommune);
            } else {
                infosElement.innerHTML += `Aucune information sur l'EPCI trouvée.`;
            }
        }
    } catch (error) {
        console.error("Erreur lors de la récupération des données :", error);
        showError("Une erreur s'est produite lors de la récupération des données. Veuillez réessayer.");
    }
}




function fetchAdresseCommune(sirenCommune) {
    const apiUrl = `https://api-lannuaire.service-public.fr/api/explore/v2.1/catalog/datasets/api-lannuaire-administration/records?where=startswith(siret,"${sirenCommune}")`;
    fetch(apiUrl).then(response => response.json()).then(data => {
        const mairieRecord = data.results.find(record => {
            const pivotData = record.pivot ? JSON.parse(record.pivot) : [];
            return (
                pivotData.some(item => item.type_service_local === "mairie") && record.nom.startsWith("Mairie - ")
            );
        });

        if (mairieRecord && mairieRecord.adresse) {
            const adresseData = JSON.parse(mairieRecord.adresse);
            const adresseMairie = `${adresseData[0].numero_voie || ''} ${adresseData[0].complement1 || ''} ${adresseData[0].complement2 || ''} ${adresseData[0].service_distribution || ''} ${adresseData[0].code_postal || ''} ${adresseData[0].nom_commune || ''}`.trim();

            // Affichage de l'adresse, validation souple
            if (adresseMairie) {
                document.getElementById('adressemairie').textContent = escapeHTML(adresseMairie);
            } else {
                console.warn("Adresse vide ou non valide :", adresseMairie);
            }

            if (mairieRecord.adresse_courriel) {
                document.getElementById('courrielmairie').textContent = escapeHTML(mairieRecord.adresse_courriel);
            }

            const siteInternetJSON = mairieRecord.site_internet;
            if (siteInternetJSON) {
                const siteInternetData = JSON.parse(siteInternetJSON);
                const siteInternet = siteInternetData.length > 0 ? siteInternetData[0].valeur : '';
                if (siteInternet) {
                    document.getElementById('sitemairie').innerHTML = `<a href="${escapeHTML(siteInternet)}" target="_blank">${escapeHTML(siteInternet)}</a>`;
                }
            }
        } else {
            infosElement.innerHTML += "Aucune information sur la Mairie trouvée.";
        }
    }).catch(error => {
        console.error("Erreur lors de la récupération des données :", error);
        showError("Une erreur s'est produite lors de la récupération des données. Veuillez réessayer.");
    });
}


function validateApiResponse(data, expectedFields) {
    return expectedFields.every(field => field in data);
}

async function fetchData(selectedCodeCommune) {
    const apiUrl = `https://geo.api.gouv.fr/communes?code=${selectedCodeCommune}&fields=code,population,codeEpci,epci,siren`;

    try {
        const response = await axios.get(apiUrl);
        const data = response.data;

        // Vérification de la structure et des champs de la réponse API
        if (data.length > 0 && validateApiResponse(data[0], ['code', 'population', 'epci', 'siren'])) {
            const codeCommune = data[0].code;
            const population = data[0].population;
            const epci = data[0].epci;
            const nomEpci = epci ? epci.nom : 'Non disponible';
            const codeEpci = data[0].codeEpci;
            const sirenCommune = data[0].siren;

            // Affichage des données de la population
            handlePopulationData(data);

            // Affichage des données de l'EPCI
            handleEpciData(data);

            // Récupération des informations sur le maire et la mairie
            handleMaireData(codeCommune);

            // Récupération des informations sur l'unité urbaine
            await handleUniteUrbaineData(codeCommune);

            // Récupération des informations sur les compétences PLU
            if (codeEpci) {
                await handlePluData(codeEpci);
            }

            // Affichage supplémentaire pour les cas particuliers de l'EPCI
            if (codeEpci && codeEpci === "200054781") {
                document.getElementById('epciInfo').textContent = `Métropole du Grand Paris – dépend d'un EPT`;
            }

        } else {
            // Si les données ne sont pas au format attendu ou sont manquantes
            showError('Aucune commune trouvée avec ce nom ou les données sont invalides.');
        }
    } catch (error) {
        console.error("Une erreur s'est produite lors de la récupération des données de l'API :", error);
        showError("Une erreur s'est produite lors de la récupération des données. Veuillez réessayer.");
    }
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
	<hr> <b>Librairies :</b>
	<ul style="list-style-type:square">
		<li><a href="https://cdnjs.com/libraries/PapaParse" target="_blank">https://cdnjs.com/libraries/PapaParse</a> – version 5.4.1</li>
  		<li><a href="https://cdn.jsdelivr.net/npm/axios/dist/" target="_blank">https://cdn.jsdelivr.net/npm/axios/dist/</a> – version 1.7.7</li>
  	</ul>
	<hr> <b>Historique :</b>
	<ul style="list-style-type:square">
 		<li>version 1.15k du 20/10/2024 : Amélioration de la sécurité</li>
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
