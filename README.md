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
		let lastSearchTimeout;
		let selectedCodeCommune;

// Fonction pour télécharger et lire les données CSV
async function fetchCsvData(url) {
    try {
        const response = await fetch(url);
        if (!response.ok) {
            throw new Error(`Erreur réseau : ${response.status} ${response.statusText}`);
        }
        const text = await response.text();
        const data = parseCsv(text); // Appel avec le séparateur par défaut ";"
        
        // Ignorer la première ligne (en-tête)
        return data.slice(1);
    } catch (error) {
        console.error("Erreur lors de la récupération du fichier CSV :", error);
        showError("Une erreur s'est produite lors de la récupération du fichier CSV.");
        return null;
    }
}

// Fonction pour parser les données CSV
function parseCsv(text, separator = ';') {
    const lines = text.trim().split('\n');
    return lines.map(line => line.split(separator));
}

// Sous-fonction pour gérer les données de la compétence PLU
async function handlePluData(codeEpci) {
    try {
        const pluResponse = await fetch('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/plu');
        if (!pluResponse.ok) {
            throw new Error(`Erreur réseau : ${pluResponse.status} ${pluResponse.statusText}`);
        }
        const pluText = await pluResponse.text();
        const lines = pluText.split('\n');
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
    } catch (error) {
        console.error("Erreur lors de la récupération des données PLU :", error);
        showError("Une erreur s'est produite lors de la récupération des données. Veuillez réessayer.");
    }
}



// Sous-fonction pour gérer les données de population
function handlePopulationData(data) {
    if (!Array.isArray(data) || data.length === 0 || typeof data[0] !== 'object' || typeof data[0].population !== 'number') {
        showError('Les données de population sont invalides ou indisponibles.');
        document.getElementById('populationInfo').textContent = 'Données non disponibles';
        return;
    }

    const population = data[0].population;
    if (Number.isInteger(population) && population >= 0) {
        // Vérifiez également que la valeur est raisonnable (par exemple, moins de 100 millions)
        if (population <= 100000000) {
            document.getElementById('populationInfo').textContent = escapeHTML(population.toString()) + ' habitants';
        } else {
            document.getElementById('populationInfo').textContent = 'Données non disponibles';
        }
    } else {
        document.getElementById('populationInfo').textContent = 'Données non disponibles';
    }
}




// Sous-fonction pour gérer les données EPCI
function handleEpciData(data) {
    if (!Array.isArray(data) || data.length === 0 || typeof data[0] !== 'object' || !data[0].epci || typeof data[0].epci.nom !== 'string' || typeof data[0].codeEpci !== 'string') {
        showError('Les données de l\'EPCI sont invalides ou indisponibles.');
        document.getElementById('epciInfo').textContent = 'Données non disponibles';
        return;
    }

    const epci = data[0].epci;
    const nomEpci = epci.nom || 'Non disponible';
    const codeEpci = data[0].codeEpci;

    if (validateText(nomEpci)) {
        document.getElementById('epciInfo').textContent = escapeHTML(nomEpci) + ' – (SIREN : ' + escapeHTML(codeEpci) + ')';
    } else {
        document.getElementById('epciInfo').textContent = 'EPCI non disponible';
    }

    if (codeEpci && codeEpci !== "200054781") {
        fetchAdresse(codeEpci, "epci");
        fetchNomEluOuPresident("president", codeEpci);
    } else {
        document.getElementById('epciInfo').textContent = `Métropole du Grand Paris – dépend d'un EPT`;
    }
}


// Sous-fonction pour gérer les informations sur le maire
function handleMaireData(codeCommune) {
    fetchNomEluOuPresident("maire", codeCommune);
    fetchAdresse(codeCommune, "mairie");
}

// Sous-fonction pour gérer les données de l'unité urbaine
async function handleUniteUrbaineData(codeCommune) {
    try {
        // Remplacement d'Axios par fetch pour la première requête
        const inseeResponse = await fetch('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/insee');
        if (!inseeResponse.ok) {
            throw new Error(`Erreur réseau : ${inseeResponse.status} ${inseeResponse.statusText}`);
        }
        const inseeText = await inseeResponse.text();
        const inseeLines = inseeText.split('\n');
        const inseeLine = inseeLines.find(line => line.startsWith(`${codeCommune},`));

        if (inseeLine) {
            const values = inseeLine.split(',');
            const numUniteUrbaine = values[1].substring(0, 5);

            // Remplacement d'Axios par fetch pour la seconde requête
            const uuResponse = await fetch('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/uu');
            if (!uuResponse.ok) {
                throw new Error(`Erreur réseau : ${uuResponse.status} ${uuResponse.statusText}`);
            }
            const uuText = await uuResponse.text();
            const uuLines = uuText.split('\n');
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
    } catch (error) {
        console.error("Une erreur s'est produite lors de la récupération des données :", error);
        showError("Une erreur s'est produite lors de la récupération des données. Veuillez réessayer.");
    }
}



function handleSearch() {
    const nomCommune = escapeHTML(communeInput.value.trim());
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

// Fonction pour afficher les messages d'erreur
function showError(message) {
    const infosElement = document.getElementById("infos");
    infosElement.textContent = message; // Afficher le message d'erreur passé en argument
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
            // Validation des données retournées par l'API
            if (!Array.isArray(data) || data.length === 0) {
                throw new Error("Les données retournées par l'API sont invalides ou vides.");
            }

            communeList.innerHTML = ''; // Vide la liste des résultats précédents
            data.forEach(function(commune) {
                // Validation des champs nécessaires pour chaque commune
                if (typeof commune.nom !== 'string' || typeof commune.codeDepartement !== 'string' || typeof commune.code !== 'string') {
                    console.warn("Données de la commune invalides : ", commune);
                    return; // Ignore les entrées invalides
                }

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
    showError("Une erreur s'est produite lors de la recherche."); // Message générique
    console.error("Détails de l'erreur :", error); // Détails spécifiques dans la console
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
    return regex.test(text.trim()) && text.length > 0 && text.length <= maxLength;
}



// Fonction d'échappement des caractères spéciaux pour éviter les injections XSS
function escapeHTML(str) {
    return str.replace(/[&<>"'`/\\]/g, function(match) {
        const escapeChars = {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
            '"': '&quot;',
            "'": '&#39;',
            '`': '&#96;',
            '/': '&#x2F;',
            '\\': '&#92;'
        };
        return escapeChars[match];
    });
}


// Fonction pour récupérer les données des élus (maire ou président)
// Fonction pour récupérer les données des élus (maire ou président)
async function fetchNomEluOuPresident(typeElu, code) {
    const csvUrlMaire = "https://static.data.gouv.fr/resources/repertoire-national-des-elus-1/20240730-125205/elus-maires.csv";
    const csvUrlPresident = "https://static.data.gouv.fr/resources/repertoire-national-des-elus-1/20240731-142441/elus-epci.csv";
    const csvUrl = typeElu === "maire" ? csvUrlMaire : csvUrlPresident;
    
    const data = await fetchCsvData(csvUrl);
    if (!data) {
        showError("Les données du CSV sont introuvables.");
        return;
    }

    let found = false;
    for (let i = 0; i < data.length; i++) {
        const row = data[i];
        const codeIndex = 4; // Colonne pour le code INSEE ou SIREN
        const fonctionIndex = 15; // Colonne pour la fonction (président)

        // Vérifier si la ligne correspond au code et, pour le président, à la bonne fonction
        if (parseInt(row[codeIndex]) === parseInt(code) &&
            (typeElu === "maire" || row[fonctionIndex] === "Président du conseil communautaire")) {

            const nomElu = row[typeElu === "maire" ? 6 : 8];
            const prenomElu = row[typeElu === "maire" ? 7 : 9];
            let sexeElu = row[typeElu === "maire" ? 8 : 10];

            // Validation des données avant l'affichage
            if (typeof nomElu === 'string' && typeof prenomElu === 'string' && validateText(nomElu) && validateText(prenomElu)) {
                sexeElu = sexeElu === "M" ? "M." : (sexeElu === "F" ? "Mme" : "");
                const infoText = typeElu === "maire" ? "nomdumaire" : "nomdupresident";
                document.getElementById(infoText).textContent = `${sexeElu} ${escapeHTML(nomElu)} ${escapeHTML(prenomElu)}`;
                found = true;
                break; // Arrêter la boucle une fois l'élu trouvé
            } else {
                console.warn("Données de l'élu invalides : ", nomElu, prenomElu);
                showError("Les informations de l'élu sont invalides.");
            }
        }
    }

    if (!found) {
        console.warn("Aucun élu correspondant trouvé pour le code :", code);
        showError("Le nom de l'élu n'a pas été trouvé.");
    }
}

async function fetchAdresse(code, type) {
    const isMairie = type === 'mairie';
    const endpoint = isMairie ? `code_insee_commune%3A%22${code}%22` : `siren%3A%22${code}%22`;
    const apiUrl = `https://api-lannuaire.service-public.fr/api/explore/v2.1/catalog/datasets/api-lannuaire-administration/records?select=pivot%2Csite_internet%2Cnom%2Cadresse_courriel%2Cadresse&where=${endpoint}&limit=100`;

    try {
        const response = await fetch(apiUrl);
        if (!response.ok) {
            throw new Error(`Erreur réseau : ${response.status} ${response.statusText}`);
        }
        const data = await response.json();

        if (!Array.isArray(data.results) || data.results.length === 0) {
            throw new Error("Données d'adresse non disponibles ou format inattendu.");
        }

        // Recherche d'un enregistrement valide pour la mairie ou l'EPCI
        const record = data.results.find(record => {
            const pivotData = record.pivot ? JSON.parse(record.pivot) : [];
            return (
                (isMairie && pivotData.some(item => item.type_service_local === "mairie") && record.nom.startsWith("Mairie - ")) || 
                (!isMairie && pivotData.some(item => item.type_service_local === "epci"))
            );
        });

        // Si un enregistrement valide est trouvé, traiter les données
        if (record && record.adresse) {
            const adresseData = JSON.parse(record.adresse);
            const adresseComplete = [
                adresseData[0].numero_voie || '',
                adresseData[0].complement1 || '',
                adresseData[0].complement2 || '',
                adresseData[0].service_distribution || '',
                adresseData[0].code_postal || '',
                adresseData[0].nom_commune || ''
            ].filter(Boolean).join(' - ');

            // Affichage de l'adresse si elle est valide
            if (adresseComplete) {
                const infoText = isMairie ? "adressemairie" : "adresseEpci";
                document.getElementById(infoText).textContent = escapeHTML(adresseComplete);
            } else {
                console.warn("Adresse vide ou non valide :", adresseComplete);
            }

            // Affichage du courriel de la mairie ou de l'EPCI si disponible
            if (record.adresse_courriel) {
                const infoText = isMairie ? "courrielmairie" : "courrielEpci";
                document.getElementById(infoText).textContent = escapeHTML(record.adresse_courriel);
            }

            // Affichage du site internet de la mairie ou de l'EPCI si disponible
            const siteInternetJSON = record.site_internet;
            if (siteInternetJSON) {
                const siteInternetData = JSON.parse(siteInternetJSON);
                const siteInternet = siteInternetData.length > 0 ? siteInternetData[0].valeur : '';
                const infoText = isMairie ? "sitemairie" : "siteEpci";
                if (siteInternet) {
                    document.getElementById(infoText).innerHTML = `<a href="${escapeHTML(siteInternet)}" target="_blank">${escapeHTML(siteInternet)}</a>`;
                }
            }
        } else {
            // Message d'erreur si aucune information n'est trouvée
            throw new Error("Aucune information sur la Mairie ou l'EPCI trouvée.");
        }
    } catch (error) {
        console.error("Erreur lors de la récupération des données :", error);
        showError("Une erreur s'est produite lors de la récupération des données. Veuillez réessayer.");
    }
}






function validateApiResponse(data, expectedFields) {
    return expectedFields.every(field => field in data);
}

async function fetchData(selectedCodeCommune) {
    const apiUrl = `https://geo.api.gouv.fr/communes?code=${selectedCodeCommune}&fields=code,population,codeEpci,epci,siren`;

    try {
        // Remplacement de axios par fetch pour l'appel API
        const response = await fetch(apiUrl);
        if (!response.ok) {
            throw new Error(`Erreur réseau : ${response.status} ${response.statusText}`);
        }
        const data = await response.json();

        // Vérification des données reçues
        if (data.length > 0 && validateApiResponse(data[0], ['code', 'population', 'epci', 'siren'])) {
            const codeCommune = data[0].code;
            const codeEpci = data[0].codeEpci;

            // Affichage des données de la population
            handlePopulationData(data);

            // Affichage des données de l'EPCI
            handleEpciData(data);

            // Récupération des informations sur le maire
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
        // Gestion des erreurs lors de la récupération des données
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

	<hr> <b>Historique :</b>
	<ul style="list-style-type:square">
 		<li>version 1.18k du 26/10/2024 : Amélioration de la sécurité</li>
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
