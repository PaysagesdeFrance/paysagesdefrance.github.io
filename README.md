<html lang="fr">

<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta http-equiv="Content-Security-Policy" content="default-src 'none'; script-src 'self'; frame-ancestors 'self'; base-uri 'none';">
	<meta http-equiv="X-Content-Type-Options" content="nosniff">
	<meta http-equiv="X-Frame-Options" content="SAMEORIGIN">
	<meta name="referrer" content="no-referrer">
	<meta http-equiv="Strict-Transport-Security" content="max-age=63072000; includeSubDomains; preload">
	<title>Recherche d'une commune</title>
	<script src="https://code.jquery.com/jquery-3.7.1.slim.min.js" integrity="sha256-kmHvs0B+OpCW5GVHUNjv9rOmY0IvSIRcf7zGUDTDQM8=" crossorigin="anonymous"></script>
	<script defer src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js" integrity="sha256-uOhwxdKyl3LxDJ+pppPIuJaqyFQO1nAePMYwTGg/69s=" crossorigin="anonymous"></script>
	<script defer src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js" integrity="sha256-KdYARiowaU79FbmEi0ykLReM0GcAknXDWjBYASERQwQ=" crossorigin="anonymous"></script>
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
	$(document).ready(function() {
		const infosElement = document.getElementById('infos');
		const communeInput = $("#communeInput");
		const communeList = $("#commune-list");
		const rechercherBtn = $("#rechercherBtn");
		let lastSearchTimeout;
		let selectedCodeCommune;

		function showError(message) {
			infosElement.textContent = message;
		}

		function hideCommuneList() {
			communeList.empty().hide();
		}
		communeInput.on("input", function() {
			var communeName = $(this).val();
			clearTimeout(lastSearchTimeout);
			lastSearchTimeout = setTimeout(function() {
				if(communeName.length >= 1) {
					fetchCommunes(communeName);
				} else {
					hideCommuneList();
				}
			}, 100);
		});

		function fetchCommunes(communeName) {
			fetch(`https://geo.api.gouv.fr/communes?nom=${communeName}&limit=13`).then(response => response.json()).then(data => {
				communeList.empty();
				data.forEach(function(commune) {
					var listItem = $("<li>").text(`${commune.nom} (${commune.codeDepartement})`);
					listItem.on("click", function() {
						selectedCodeCommune = commune.code;
						communeInput.val(commune.nom);
						hideCommuneList();
						infosElement.innerHTML = '';
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
						resultatCommune.innerHTML = `<h2>– ${commune.nom} (${commune.codeDepartement}) – code INSEE ${selectedCodeCommune}</h2>`;
						if(resultatCommune.innerHTML.trim() !== "") {
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
			if(!communeInput.is(event.target) && !communeList.is(event.target) && communeList.has(event.target).length === 0) {
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

		function fetchNomEluOuPresident(typeElu, code) {
			const csvUrlMaire = "https://www.data.gouv.fr/fr/datasets/r/2876a346-d50c-4911-934e-19ee07b0e503";
			const csvUrlPresident = "https://www.data.gouv.fr/fr/datasets/r/41d95d7d-b172-4636-ac44-32656367cdc7";
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
							if(sexeElu === "M") {
								sexeElu = "M.";
							} else if(sexeElu === "F") {
								sexeElu = "Mme";
							}
							const infoText = typeElu === "maire" ? "nomdumaire" : "nomdupresident";
							document.getElementById(infoText).textContent = `${sexeElu} ${nomElu} ${prenomElu}`;
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
					return(
						(isMairie && pivotData.some(item => item.type_service_local === "mairie") && record.nom.startsWith("Mairie - ")) || (!isMairie && pivotData.some(item => item.type_service_local === "epci")));
				});
				if(mairieRecord) {
					const adresseData = JSON.parse(mairieRecord.adresse);
					const adresseMairie = `${adresseData[0].numero_voie} – ${adresseData[0].complement1} – ${adresseData[0].complement2} – ${adresseData[0].service_distribution} – ${adresseData[0].code_postal} ${adresseData[0].nom_commune}`;
					const infoText = type === "mairie" ? "adressemairie" : "adresseEpci";
					document.getElementById(infoText).textContent = `${adresseMairie}`;
					if(mairieRecord.adresse_courriel) {
						const infoText = type === "mairie" ? "courrielmairie" : "courrielEpci";
						document.getElementById(infoText).textContent = `${mairieRecord.adresse_courriel}`;
					}
					const siteInternetJSON = mairieRecord.site_internet;
					if(siteInternetJSON) {
						const siteInternetData = JSON.parse(siteInternetJSON);
						const siteInternet = siteInternetData.length > 0 ? siteInternetData[0].valeur : '';
						const infoText = type === "mairie" ? "sitemairie" : "siteEpci";
						document.getElementById(infoText).innerHTML = `<a href="${siteInternet}" target="_blank">${siteInternet}</a>`;
					}
				} else {
					if(isMairie) {
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
					return(pivotData.some(item => item.type_service_local === "mairie") && record.nom.startsWith("Mairie - "));
				});
				if(mairieRecord) {
					const adresseData = JSON.parse(mairieRecord.adresse);
					const adresseMairie = `${adresseData[0].numero_voie} – ${adresseData[0].complement1} – ${adresseData[0].complement2} – ${adresseData[0].service_distribution} – ${adresseData[0].code_postal} ${adresseData[0].nom_commune}`;
					document.getElementById('adressemairie').textContent = `${adresseMairie}`;
					if(mairieRecord.adresse_courriel) {
						document.getElementById('courrielmairie').textContent = `${mairieRecord.adresse_courriel}`;
					}
					const siteInternetJSON = mairieRecord.site_internet;
					if(siteInternetJSON) {
						const siteInternetData = JSON.parse(siteInternetJSON);
						const siteInternet = siteInternetData.length > 0 ? siteInternetData[0].valeur : '';
						document.getElementById('sitemairie').textContent = `${siteInternet}`;
					}
				} else {
					infosElement.innerHTML += "Aucune information sur la Mairie trouvée.";
				}
			}).catch(error => {
				console.error("Erreur lors de la récupération des données :", error);
				showError("Une erreur s'est produite lors de la récupération des données. Veuillez réessayer.");
			});
		}

		function fetchData(selectedCodeCommune) {
			const apiUrl = `https://geo.api.gouv.fr/communes?code=${selectedCodeCommune}&fields=code,population,codeEpci,epci,siren`;
			axios.get(apiUrl).then(response => response.data).then(data => {
				if(data.length > 0) {
					const codeCommune = data[0].code;
					const population = data[0].population;
					const epci = data[0].epci;
					const nomEpci = epci ? epci.nom : 'Non disponible';
					const codeEpci = data[0].codeEpci;
					sirenCommune = data[0].siren;
					document.getElementById('populationInfo').textContent = `${population} habitants`;
					document.getElementById('epciInfo').textContent = `${nomEpci} – (SIREN : ${codeEpci})`;
					fetchNomEluOuPresident("maire", codeCommune);
					fetchAdresseData(codeCommune, "mairie");
					if(codeEpci !== "200054781") {
						fetchAdresseData(codeEpci, "epci");
						fetchNomEluOuPresident("president", codeEpci);
					} else {
						document.getElementById('epciInfo').textContent = `Métropole du Grand Paris – dépend d'un EPT`;
					}
					axios.get('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/plu').then(response => response.data).then(text => {
						const lines = text.split('\n');
						const line = lines.find(line => line.match(`^${codeEpci},`));
						if(line) {
							const uuValues = line.split(',');
							const numAssocie = uuValues[1].toString();
							if(codeEpci !== "200054781") {
								let message = "";
								if(numAssocie === "0") {
									message = "non";
								} else if(numAssocie === "1") {
									message = "oui";
								} else {
									message = "Valeur inconnue";
								}
								document.getElementById('competencePLU').textContent = message;
							}
						}
					});
					axios.get('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/insee').then(response => response.data).then(text => {
						const lines = text.split('\n');
						const line = lines.find(line => line.match(`^${codeCommune},`));
						if(line) {
							const values = line.split(',');
							let numUniteUrbaine = values[1].toString().substring(0, 5);
							axios.get('https://raw.githubusercontent.com/PaysagesdeFrance/pdf/main/uu').then(response => response.data).then(text => {
								const uuLines = text.split('\n');
								const uuLine = uuLines.find(uuLine => uuLine.includes(`${numUniteUrbaine},`));
								if(uuLine) {
									const uuValues = uuLine.split(',');
									const numAssocie = uuValues[1].toString();
									if(numAssocie <= 5) {
										document.getElementById('popUrbaineInfo').textContent = `inférieure à 100000 habitants`;
									} else if(numAssocie == 8) {
										document.getElementById('popUrbaineInfo').textContent = `unité urbaine de Paris`;
									} else if(numAssocie == 6 || numAssocie == 7) {
										document.getElementById('popUrbaineInfo').textContent = `supérieure à 100000 habitants`;
									} else {
										document.getElementById('popUrbaineInfo').textContent = `Aucune condition spécifiée`;
									}
								} else {
									document.getElementById('popUrbaineInfo').textContent = `hors unité urbaine`;
								}
							}).catch(error => {
								console.error("Une erreur s'est produite lors de la récupération des données du fichier uu :", error);
								showError("Une erreur s'est produite lors de la récupération des données. Veuillez réessayer.");
							});
						}
					}).catch(error => {
						console.error("Une erreur s'est produite lors de la récupération des données du fichier insee :", error);
						showError("Une erreur s'est produite lors de la récupération des données. Veuillez réessayer.");
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
		<li>(2) informations mises à jour manuellement – valable au 1er janvier 2023 – source : <a href="https://www.insee.fr/fr/information/4802589" target="_blank">https://www.insee.fr/fr/information/4802589</a></li>
		<li>(3) OpenData gouvernemental : Ministère de l'Intérieur et des Outre-Mer – <a href="https://www.data.gouv.fr/fr/datasets/repertoire-national-des-elus-1/" target="_blank">https://www.data.gouv.fr/fr/datasets/repertoire-national-des-elus-1/</a></li>
		<li>(4) API gouvernementale : <a href="https://api-lannuaire.service-public.fr/explore/dataset/api-lannuaire-administration" target="_blank">https://api-lannuaire.service-public.fr/explore/dataset/api-lannuaire-administration</a></li>
		<li>(5) informations mises à jour manuellement – valable au 10 octobre 2023 – source : <a href="https://www.banatic.interieur.gouv.fr/" target="_blank">https://www.banatic.interieur.gouv.fr/</a></li>

	</ul>
	<hr> <b>Librairies :</b>
	<ul style="list-style-type:square">
		<li><a href="https://releases.jquery.com/" target="_blank">https://releases.jquery.com/</a> – version 3.7.1</li>
		<li><a href="https://cdnjs.com/libraries/PapaParse" target="_blank">https://cdnjs.com/libraries/PapaParse</a> – version 5.4.1</li>
	</ul>
	<hr> <b>Historique :</b>
	<ul style="list-style-type:square">
 		<li>version 1.04n du 17/03/2024 : Implantation Content Security Policy header, Subresource integrity, X-Content-Type-Options header, X-Frame-Options header, referrer-policy header</li>
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
