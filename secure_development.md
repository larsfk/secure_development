# Sikrere utvikling
Dette dokumentet skal gi utviklere en oversikt over hvilke verktøy, rammeverk og grep de kan gjøre i hverdagen for å sørge for at koden de skriver er skrevet på en sikker måte. Forhåpentligvis vil utviklere som leser dette dokumentet bli motivert til å tenke mer sikkerhet og kanskje endre måten de utvikler på. 

Håper også at flere ønsker å bidra inn på dette dokumentet for å gjøre det bedre og mer komplett. Dette er tross alt et stort tema. 

# Verktøy

## SonarQube
SonarQube [sonarqube.org](https://www.sonarqube.org/) er et verktøy som skanner koden din og finner bugs, kodeforbedringer, svakheter og teknisk beregner teknisk gjeld. SonarQube kan kjøres lokalt eller settes opp som en service i Azure eller AWS. 

SonarQube kan også integreres med CI/CD-pipelines via Jenkins eller Azure DevOps for å få ut en SonarQube-rapport for det som blir pushet ut til de ulike miljøene. SonarQube kan også kobles til brancher for å gjøre branch-scanning før en Pull Request for å finne feil, bugs og svakheter før de blir pushet inn i en pipeline. 


### Hvordan sette opp SonarQube lokalt

Pre-requirements:
- Installert docker (docker for windows feks) [https://docs.docker.com/docker-for-windows/install/](https://docs.docker.com/docker-for-windows/install/).
  - Sjekk at docker er installert ved å åpne en kommando-shell ([cmder](https://cmder.net/) eller git bash feks) og skriv `docker --help`

Kan kjøre uten docker også, men det er litt mer trøblete: [Getting started in two miuntes.](https://docs.sonarqube.org/latest/setup/get-started-2-minutes/).
  
Når docker er installert kan vi begynne å sette opp SonarQube lokalt.

1. Last ned docker-image for SonarQube:
`docker pull sonarqube`

2. Kjør SonarQube-imaget via docker: 
`docker run -d --name sonarqube -p 9000:9000 sonarqube`

3. Besøk [http://localhost:9000](http://localhost:9000) for å se at instansen kjører (kan ta ett minutt eller to før den er oppe)
   1. Sjekk evt `docker ps` for se at konteineren kjører. 

Obs! brukernavn og passord lokalt er `admin` og `admin`.

Nå som SonarQube kjører kan vi gå i gang med scanningen.

### Hvordan scannet et prosjekt

Pre-requirements:
- SonarQube (forrige steg) må være installert og kjørende
- Javascript m.m: Last ned nyeste versjon av [SonarScanner for JS, TS etc](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/#AnalyzingwithSonarQubeScanner-Use)
- C#: Last ned nyeste versjon av [SonarScanner for MSBuild](https://sonarcloud.io/documentation/analysis/scan/sonarscanner-for-msbuild/)
- Java: bruker maven eller gradle (ikke testet)

Når SonarScanner er lastet ned så må SonarScanner pakkes ut og legges til et i PATH (Environment Variables på Windows). Kan derfor være kjekt å pakke ut SonarScanner.zip et eller annet lurt sted. 

**La oss scanne et prosjekt:**
1. I ditt favoritt shell (cmder feks): `SonarScanner.MSBuild.exe /h` og/eller `sonar-scanner.bat -v` (sonar-scanner -v i UNIX). 
   1. MERK! Etter å lagt til sonar-scanner.bat/SonarScanner.MSBuild.exe i PATH må alle shell restartes. 
2. Finn et C#/Java-prosjekt og flytt deg inn til prosjektets toppnivå via terminalen (shell). 
3. Før vi scanner må vi lage et prosjekt i SonarQube [http://localhost:9000](localhost:9000) (se bilde under).  
   
![Step 3: Create prosject](https://github.com/larsfk/secure_development/blob/master/images/sonarqube_create_new_project.PNG)

4. Fyll ut prosjektnavn og prosjektnøkkel (kan godt være det samme).
5. Generere en token. Eks: project-name-backend-token
6. Kopier tokenen som er vist til et lurt sted, spar på denne!
7. Velg så om du ønsker å gjøre en scan i et java- eller C#-prosjekt eller "annet".
8. Kjør så kommandoen som er vist på skjermen. 

C#:
```
SonarScanner.MSBuild.exe begin /k:"INSERT_PROJECT-KEY" /d:sonar.host.url="http://localhost:9000" /d:sonar.login="INSERT_TOKEN"
MsBuild.exe /t:Rebuild
SonarScanner.MSBuild.exe end /d:sonar.login="TOKEN"
```

Other:
```
sonar-scanner.bat -D"sonar.projectKey=INSERT_HER" -D"sonar.sources=." -D"sonar.host.url=http://localhost:9000" -D"sonar.login=INSERT_TOKEN"
```

Maven:
```
mvn sonar:sonar \
  -Dsonar.projectKey=INSERT_PROJECTKEY \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=INSERT_TOKEN
```

Gradle:
Declare org.sonarqube plugin in build.gradle:
```
plugins {
  id "org.sonarqube" version "2.7"
}
```
```
./gradlew sonarqube \
  -Dsonar.projectKey=INSERT_PROJECTKEY \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=INSERT_TOKEN
```


Etter å kjørt scannen på prosjektet vil prosjektet i SonarQube oppdateres med de nye funnene.


## OWASP Dependency Check
Nå som man har en rapport på hvordan koden i prosjektet er, så er det nyttig å utvide rapporten til å inkludere en sjekk rundt OWASP sine kjente sårbarheter. For å gjøre dette kan man bruke OWASP Dependency Check [https://github.com/jeremylong/DependencyCheck](https://github.com/jeremylong/DependencyCheck). Dette er en sjekk som kjøres utenom SonarQube og kan fint brukes alene, men den kan også inkluderes i SonarQube ved å bruke en plugin. 

Først skal vi kjøre en Dependency Check for prosjektet vårt, før vi inkluderer sjekken i SonarQube-prosjektet. 

### Installere og kjøre OWASP Dependency Check
1. OWASP Dependency Check CLI kan lastes ned her: [https://dl.bintray.com/jeremy-long/owasp/dependency-check-5.2.1-release.zip](https://dl.bintray.com/jeremy-long/owasp/dependency-check-5.2.1-release.zip).
2. Legg så til dependency-check.bat til i PATH.
3. Åpne/restart ditt favoritt shell og kjør `dependency-check.bat -v` for å se at den er lagt til riktig.
4. Nå kan vi kjøre Dependency Check-en:

```dependency-check.bat --project "Name Of Project" --scan "PROJECT_ROOT"``` (or use . after --scan for current folder)

Nå generes det en HTML-fil som inneholder en rapport om OWASP Vulnerabilites. Samme rapport kan generes i XML og dermed inkluderes i SonarQube ved hjelp av en plugin. 

### OWASP Dependency Check i SonarQube
For å få OWASP-rapporten inn i SonarQube må følgende gjøres:
1. Generere en rapport i XML-format for prosjektet (se forrige seksjon)
2. Last ned `sonar-dependency-check-plugin-xxx.jar` [https://github.com/SonarSecurityCommunity/dependency-check-sonar-plugin ](https://github.com/SonarSecurityCommunity/dependency-check-sonar-plugin/releases/tag/1.2.5)

Denne .jar-filen skal nå legges til som en plugin i SonarQube. Dette gjøres ved å legge filen inn i `/extensions/plugins` i SonarQube-service-mappen. Dersom du kjører docker må du SSHe deg inn på kontaineren og kopiere inn filene der. Hvis du kjører SonarQube-serveren lokalt så kan du bare kopiere .jar-filen rett inn i mappen du kjører serveren fra. 

**For docker:**
1. `docker ps` for å få oversikt over alle kontainerne som kjører.
![docker ps](https://github.com/larsfk/secure_development/blob/master/images/docker_cp.PNG)
2. `docker exec -it CONTAINER_ID /bin/bash` (630, trenger ikke hele, for mitt tilfellet)
![docker exec](https://github.com/larsfk/secure_development/blob/master/images/docker_exec.PNG)
3. `ls` viser blant annet mappen `extensions` som inneholder `plugins`.
![ls](https://github.com/larsfk/secure_development/blob/master/images/docker_ls.PNG)
4. Åpne en ny terminal/shell for å kopiere filer fra lokalt til docker
5. Naviger til `sonar-dependency-check-plugin-xxx.jar`
6. Skriv så `docker cp sonar-dependency-check-plugin-xxx.jar CONTAINER_ID:opt/sonarqube/extensions/plugins/sonar-dependency-check-plugin-xxx.jar`
![docker cp](https://github.com/larsfk/secure_development/blob/master/images/docker_cp.PNG)
7. Se så at filen ligger i docker kontaineren under extensions/plugins. 
8. Gå inn på localhost:9000 og restart serveren (Administration -> System -> Restart Server)
9. Nå kan SonarScanner kjøres på nytt, men legg til følgende "flagg":
`/d:"sonar.dependencyCheck.reportPath=<FILL_PATH_TO_REPORT_XML_IN_HERE>"` (C#) eller `/-D:"sonar.dependencyCheck.reportPath=<FILL_PATH_TO_REPORT_XML_IN_HERE>"` (Other)


For mer info: https://maartenderaedemaeker.be/2017/07/27/using-owasp-dependency-check/ 


## OWASP Cheat Sheet
En QnA og guide til sikrere utvikling av WebApps.
https://cheatsheetseries.owasp.org/ 


## AttackFlow Secure Coding Assistant (Visual Studio)
AttackFlow Extension er en plugin til forskjellige IDE-er. Den kan brukes til å scanne filer man skrive for å finne kjente feil og bugs. 

Link: [https://www.attackflow.com/Extension ](https://marketplace.visualstudio.com/items?itemName=vs-publisher-1169494.AttackFlow)
AttackFlow har også andre versjon som koster penger. https://www.attackflow.com


## Security Code Scan
Verktøy for statisk sikkerhetsscan av kode.
Plugin for .NET: https://github.com/security-code-scan/security-code-scan and https://security-code-scan.github.io/.



## SonarLint
SonarQube sin egen linter: https://www.sonarlint.org/ 


# Awesome-pentest
https://github.com/enaqx/awesome-pentest 
