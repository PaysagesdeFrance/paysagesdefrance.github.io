<html lang="fr">

<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta http-equiv="Content-Security-Policy" content="default-src 'none'; script-src 'self' https://code.jquery.com https://cdnjs.cloudflare.com https://cdn.jsdelivr.net; connect-src 'self' https://geo.api.gouv.fr https://api-lannuaire.service-public.fr; frame-ancestors 'none';">
	<meta http-equiv="X-Content-Type-Options" content="nosniff">
	<meta name="referrer" content="strict-origin">
	<meta http-equiv="Strict-Transport-Security" content="max-age=63072000; includeSubDomains; preload">
	<title>Recherche d'une commune</title>
	<script src="https://code.jquery.com/jquery-3.7.1.slim.min.js" integrity="sha384-5AkRS45j4ukf+JbWAfHL8P4onPA9p0KwwP7pUdjSQA3ss9edbJUJc/XcYAiheSSz" crossorigin="anonymous"></script>
	<script defer src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js" integrity="sha384-D/t0ZMqQW31H3az8ktEiNb39wyKnS82iFY52QPACM+IjKW3jDUhyIgh2PApRqJZs" crossorigin="anonymous"></script>
	<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js" integrity="sha384-BSfV8fZD1qtU+7KBLfwU6gmcvPJpmwSoXe28kfEv6tS2XTcI4RYmu9GIrKLrhA4y" crossorigin="anonymous"></script>
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
		const infosElement = document.getElementById('infos');
		const communeInput = document.getElementById("communeInput");
		const communeList = $("#commune-list");
		const rechercherBtn = $("#rechercherBtn");
		let lastSearchTimeout;
		let selectedCodeCommune;

function showError(message) {
    infosElement.textContent = "Une erreur est survenue. Veuillez réessayer.";
    console.error("Détails de l'erreur :", message); // Les détails restent dans la console
}


		function hideCommuneList() {
communeList.empty().hide();
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

		communeInput.on("input", debounce(function() {
    var communeName = this.value;
    if (communeName.length >= 1) {
        fetchCommunes(communeName);
    } else {
        hideCommuneList();
    }
}, 300));

		function fetchCommunes(communeName) {
			fetch(`https://geo.api.gouv.fr/communes?nom=${communeName}&limit=13`).then(response => response.json()).then(data => {
				communeList.empty();
				data.forEach(function(commune) {
					var listItem = $("<li>").text(`${commune.nom} (${commune.codeDepartement})`);
					listItem.on("click", function() {
						selectedCodeCommune = commune.code;
						communeInput.value = commune.nom;
						hideCommuneList();
						infosElement.textContent = '';
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
						$("table").css("display", "none");
						
      						const h2Element = document.createElement('h2');
	    					h2Element.textContent = `– ${commune.nom} (${commune.codeDepartement}) – code INSEE ${selectedCodeCommune}`;
	  					resultatCommune.textContent = ''; // Effacer le contenu précédent
						resultatCommune.appendChild(h2Element);
	    					
						if(resultatCommune.textContent.trim() !== "") {
							rechercherBtn.focus();
						}
					});
					communeList.append(listItem);
				});
				communeList.show();
			}).catch(error => {
				showError("Une erreur s'est produite lors de la recherche des communes. Veuillez réessayer.");
				console.error("Une erreur s'est produite lors de la récupération du fichier CSV :", error);
			});
		}
$(document).on("click", function(event) {
    if (!communeInput.is(event.target) && !communeList.is(event.target) && communeList.has(event.target).length === 0) {
        hideCommuneList();
    }
});
		rechercherBtn.on("click", function() {
			const nomCommune = communeInput.val().trim();
			infosElement.textContent = '';
			if(selectedCodeCommune) {
				fetchData(selectedCodeCommune);
				$("table").css("display", "table");
			} else {
				showError('Veuillez entrer le nom d\'une commune.');
			}
		});


function validateText(text) {
    const regex = /^[A-Za-zÀ-ÖØ-öø-ÿ0-9 ',-]+$/; // Permet les lettres, les chiffres, les espaces, les apostrophes, les virgules et les tirets
    return regex.test(text);
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
                const codeIndex = typeElu === "maire" ? 4 : 4;
                const fonctionIndex = typeElu === "maire" ? 15 : 15;
                if(parseInt(data[i][codeIndex]) === parseInt(code) && (typeElu === "maire" || data[i][fonctionIndex] === "Président du conseil communautaire")) {
                    const nomElu = data[i][typeElu === "maire" ? 6 : 8];
                    const prenomElu = data[i][typeElu === "maire" ? 7 : 9];
                    let sexeElu = data[i][typeElu === "maire" ? 8 : 10];

                    // Validation et échappement des données
                    if (validateText(nomElu) && validateText(prenomElu)) {
                        if (sexeElu === "M") {
                            sexeElu = "M.";
                        } else if (sexeElu === "F") {
                            sexeElu = "Mme";
                        }
                        const infoText = typeElu === "maire" ? "nomdumaire" : "nomdupresident";
                        document.getElementById(infoText).textContent = `${sexeElu} ${escapeHTML(nomElu)} ${escapeHTML(prenomElu)}`;
                    } else {
                        console.warn("Données de l'élu invalides : ", nomElu, prenomElu);
                    }
                    break;
                }
            }
        },
        error: function(error) {
            showError("Une erreur s'est produite lors de la récupération du fichier CSV. Veuillez réessayer.");
            console.error("Une erreur s'est produite lors de la récupération du fichier CSV :", error);
        }
    });
}

function fetchAdresseData(code, type) {
    const isMairie = type === 'mairie';
    const endpoint = isMairie ? `code_insee_commune%3A%22${code}%22` : `siren%3A%22${code}%22`;
    const apiUrl = `https://api-lannuaire.service-public.fr/api/explore/v2.1/catalog/datasets/api-lannuaire-administration/records?select=pivot%2Csite_internet%2Cnom%2Cadresse_courriel%2Cadresse&where=${endpoint}&limit=100`;
    fetch(apiUrl).then(response => response.json()).then(data => {
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
    }).catch(error => {
        console.error("Erreur lors de la récupération des données :", error);
        showError("Une erreur s'est produite lors de la récupération des données. Veuillez réessayer.");
    });
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

function fetchData(selectedCodeCommune) {
    const apiUrl = `https://geo.api.gouv.fr/communes?code=${selectedCodeCommune}&fields=code,population,codeEpci,epci,siren`;
    axios.get(apiUrl).then(response => response.data).then(data => {
        if (data.length > 0 && validateApiResponse(data[0], ['code', 'population', 'epci', 'siren'])) {
            const codeCommune = data[0].code;
            const population = data[0].population;
            const epci = data[0].epci;
            const nomEpci = epci ? epci.nom : 'Non disponible';
            const codeEpci = data[0].codeEpci;
            sirenCommune = data[0].siren;

            // Validation et échappement des données avant insertion dans le DOM
            if (Number.isInteger(population)) {
                document.getElementById('populationInfo').textContent = escapeHTML(population.toString()) + ' habitants';
            } else {
                document.getElementById('populationInfo').textContent = 'Données non disponibles';
            }

            if (nomEpci && validateText(nomEpci)) {
                document.getElementById('epciInfo').textContent = escapeHTML(nomEpci) + ' – (SIREN : ' + escapeHTML(codeEpci) + ')';
            } else {
                document.getElementById('epciInfo').textContent = 'EPCI non disponible';
            }

            // Récupération des informations complémentaires sur les élus et les adresses

            
            fetchNomEluOuPresident("maire", codeCommune);
            fetchAdresseData(codeCommune, "mairie");

            if (codeEpci && codeEpci !== "200054781") {
                fetchAdresseData(codeEpci, "epci");
                fetchNomEluOuPresident("president", codeEpci);
            } else {
                document.getElementById('epciInfo').textContent = `Métropole du Grand Paris – dépend d'un EPT`;
            }

            // Récupération des informations sur les compétences PLU
            axios.get('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/plu').then(response => response.data).then(text => {
                const lines = text.split('\n');
                const line = lines.find(line => line.match(`^${codeEpci},`));
                if (line) {
                    const uuValues = line.split(',');
                    const numAssocie = uuValues[1].toString();
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
            }).catch(error => {
                console.error("Erreur lors de la récupération des données PLU :", error);
                showError("Une erreur s'est produite lors de la récupération des données PLU. Veuillez réessayer.");
            });

            // Récupération des informations sur les unités urbaines
            axios.get('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/insee').then(response => response.data).then(text => {
                const lines = text.split('\n');
                const line = lines.find(line => line.match(`^${codeCommune},`));
                if (line) {
                    const values = line.split(',');
                    let numUniteUrbaine = values[1].toString().substring(0, 5);
                    axios.get('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/uu').then(response => response.data).then(text => {
                        const uuLines = text.split('\n');
                        const uuLine = uuLines.find(uuLine => uuLine.includes(`${numUniteUrbaine},`));
                        if (uuLine) {
                            const uuValues = uuLine.split(',');
                            const numAssocie = uuValues[1].toString();
                            let populationUrbainMessage = "";
                            if (numAssocie <= 5) {
                                populationUrbainMessage = "inférieure à 100000 habitants";
                            } else if (numAssocie == 8) {
                                populationUrbainMessage = "unité urbaine de Paris";
                            } else if (numAssocie == 6 || numAssocie == 7) {
                                populationUrbainMessage = "supérieure à 100000 habitants";
                            } else {
                                populationUrbainMessage = "Aucune condition spécifiée";
                            }
                            document.getElementById('popUrbaineInfo').textContent = escapeHTML(populationUrbainMessage);
                        } else {
                            document.getElementById('popUrbaineInfo').textContent = "hors unité urbaine";
                        }
                    }).catch(error => {
                        console.error("Erreur lors de la récupération des données de l'unité urbaine :", error);
                        showError("Une erreur s'est produite lors de la récupération des données. Veuillez réessayer.");
                    });
                } else {
                    document.getElementById('popUrbaineInfo').textContent = "Information non disponible";
                }
            }).catch(error => {
                console.error("Erreur lors de la récupération des données INSEE :", error);
                showError("Une erreur s'est produite lors de la récupération des données INSEE. Veuillez réessayer.");
            });
        } else {
            showError('Aucune commune trouvée avec ce nom.');
        }
    }).catch(error => {
        console.error("Une erreur s'est produite lors de la récupération des données de l'API :", error);
        showError("Une erreur s'est produite lors de la récupération des données. Veuillez réessayer.");
    });
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
		<li><a href="https://releases.jquery.com/" target="_blank">https://releases.jquery.com/</a> – version 3.7.1</li>
		<li><a href="https://cdnjs.com/libraries/PapaParse" target="_blank">https://cdnjs.com/libraries/PapaParse</a> – version 5.4.1</li>
  		<li><a href="https://cdn.jsdelivr.net/npm/axios/dist/" target="_blank">https://cdn.jsdelivr.net/npm/axios/dist/</a> – version 1.7.7</li>
  	</ul>
	<hr> <b>Historique :</b>
	<ul style="list-style-type:square">
 		<li>version 1.14m du 19/10/2024 : Amélioration de la sécurité</li>
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
