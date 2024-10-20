# Release workbook
---

### Klargjøring
* Sørg for at du har u/p til alle servere
* Få NO filen fra helsesdirektoratet (tidligere: Last ned release fra [mlds](https://mlds.ihtsdotools.org/#/viewReleases))
---

### Smoketest [Demo](https://demo.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/)
* Deaktivere DailyBuild
  >**PUT /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >Request body:  
  ```
  {
    "name": "Norwegian Edition",
    "owner": "The Norwegian Directorate of Health",
    "countryCode": "no",
    "defaultLanguageCode": "no",
    "dailyBuildAvailable": false
  }
  ```
  >*Execute*  
* Verifiser at dailyBuild er av og at SNOMEDCT-NO har riktig avhengighet til den internasjonale utgaven
  >**GET /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
  >Response body:  
  ```
  {
    .
    .
    "dependantVersionEffectiveTime": YYYYMMDD,
    "dailyBuildAvailable": false,
    .
    .
  }
  ```
* Rollback DailyBuild 
  >**POST /codesystems/{shortname}daily-build/rollback**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
* Sjekk at *Authoring Stats* står til 0. Hvis ikke gjør en ny rollback.
  >**GET /{branch}/authoring-stats**  
  >branch: MAIN/SNOMEDCT-NO  
  >*Execute*  
  ```
  {
    "newConceptsCount": 0,
    "inactivatedConceptsCount": 0,
    "reactivatedConceptsCount": 0,
    "changedFsnCount": 0,
    "inactivatedSynonymsCount": 0,
    "newSynonymsForExistingConceptsCount": 0,
    "reactivatedSynonymsCount": 0,
    "executionTime": 1725441442988,
    "title": "Authoring changes since last release"
  }
  ```

* Lag en import job
  >**POST /imports**  
  >Request body:  
  ```
  {
    "type": "SNAPSHOT", 
    "branchPath": "MAIN/SNOMEDCT-NO", 
    "createCodeSystemVersion": true  
  }
  ```
  >*Execute*  
  >*Kopier importId*  

* Sjekk status på import job
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "WAITING_FOR_FILE"
    .
    .
  }
  ```
* Importer fil
  >**GET /imports/{importId}/archive**  
  >ImportId:*importId fra tidligere steg*  
  >file: *choose file* - velg zip filen du lastet ned fra mlds i første steg  
  >*Execute*  
* Vent 5ish min og ta en ny sjekk av status på import job som over. Hvis status er "RUNNING", vent litt til, gå videre når status er "COMPLETED"
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "COMPLETED"
    .
    .
  }
  ```
* Gi beskjed til helsedirektoratet slik at de kan teste den norske ekstensjonen
---

### Importer i [DailyBuild](https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/) for eksport av derivater

* Sjekk status på branches for å se hvordan det ser ut før første import
>**GET** branches  
>*Execute*  
##### !!! Delen over er for min egen del 2024-09-15
* Deaktivere dailyBuild
  >**PUT /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >Request body:  
  ```
  {
    "name": "Norwegian Edition",
    "owner": "The Norwegian Directorate of Health",
    "countryCode": "no",
    "defaultLanguageCode": "no",
    "dailyBuildAvailable": false
  }
  ```
  >*Execute*  
* Verifiser at dailyBuild er av og at SNOMEDCT-NO har riktig avhengighet til den internasjonale utgaven
  >**GET /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
  ```
  {
    .
    .
    "dependantVersionEffectiveTime": YYYYMMDD,
    "dailyBuildAvailable": false,
    .
    .
  }
  ```
* Rollback DailyBuild 
  >**POST /codesystems/{shortname}daily-build/rollback**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
* Sjekk at *Authoring Stats* står til 0. Hvis ikke gjør en ny rollback.
  >**GET /{branch}/authoring-stats**  
  >branch: MAIN/SNOMEDCT-NO  
  >*Execute*  
  ```
  {
    "newConceptsCount": 0,
    "inactivatedConceptsCount": 0,
    "reactivatedConceptsCount": 0,
    "changedFsnCount": 0,
    "inactivatedSynonymsCount": 0,
    "newSynonymsForExistingConceptsCount": 0,
    "reactivatedSynonymsCount": 0,
    "executionTime": 1725441442988,
    "title": "Authoring changes since last release"
  }
  ```
* Lag en import job
  >**POST /imports**
  Request body:
  ```
  {
    "type": "SNAPSHOT", 
    "branchPath": "MAIN/SNOMEDCT-NO", 
    "createCodeSystemVersion": "true"
  }
  ```
  >*Execute*  
  >Response body:  
  ```
  {
    .
    .
    location: https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/imports/[LANG importId TALLSTRENG HER]
    .
    .
  }
  ```
  >*Kopier importId*  

* Sjekk status på import job
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "WAITING_FOR_FILE"
    .
    .
  }
  ```
* Importer fil
  >**GET /imports/{importId}/archive**  
  >ImportId:*importId fra tidligere steg*  
  >file: *choose file* - velg zip filen du lastet ned fra mlds i første steg  
  >*Execute*  
* Vent 5ish min og ta en ny sjekk av status på import job som over. Hvis status er "RUNNING", vent litt til
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "COMPLETED"
    .
    .
  }
  ```
* ##### !!! Denne er for min egen del 2024-09-15
* Sjekk status på branches for å se hvordan det ser ut etter første import
>**GET** branches  
>*Execute*  
---

### Generer og hent rf2 pakker
##### !!! 2024-09-15 Vi måtte sette codesystem.all.latest-version.allow-future=true i /opt/snowstorm/application.properties på DB for å få sett riktig release
* Generer rf2 pakker for derivatene fra [refset-editor](https://refset.terminologi.helsedirektoratet.no)
  - Log in
  - Velg ***Release*** oppe til høyre
  - Velg organisasjonsenhet og eventuellt fjern referansesett fra utvalg
    > Direktoratet for e-helse - Diagnoser: Fjern hake for ICPC2 (importeres via backend API)  
    > Direktoratet for e-helse - Legemidler: Alle markert  
    > International Council of Nurses (ICN): Alle markert  
  - RelaseCodeSystem NewDependantVersion: 20YY-MM-15
  - Versjonsdato: 20YYMM15
  - *Lag ny versjon i snowstorm dailybuild og eksporter pakke klar til produksjon*: markert
  - La resten være som det er og trykk på ***Lag ny versjon***
  - Vær tålmodig! Det kan ta ganske lang tid å få generert pakkene
* Last ned de zippede rf2 pakkene fra Azure
  - portal - storage accounts - stfsflsoutputsprodnoe1 - containers - refset-release-dev
---

### Importere norsk ekstensjon og derivater i [prod](https://snowstorm.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/)

* Verifiser at SNOMEDCT-NO har riktig avhengighet til den internasjonale utgaven
  >**GET /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
  ```
  {
    .
    .
    "dependantVersionEffectiveTime": YYYYMMDD,
    "dailyBuildAvailable": false,
    .
    .
  }
  ```
* Start med norsk ekstensjon, gjenta deretter prosedyren under med rf2 pakkene lastet ned fra Azure (Diagnoser, Legemidler og ICN):

* *!!!-- prosedyre start --!!!*

* Lag en import job
  >**POST /imports**
  Request body:
  ```
  {
    "type": "SNAPSHOT", 
    "branchPath": "MAIN/SNOMEDCT-NO", 
    "createCodeSystemVersion": "false"  
  }
  ```
  >*Execute*  
  >Response body:  
  ```
  {
    .
    .
    location: https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/imports/[LANG importId TALLSTRENG HER]
    .
    .
  }
  ```
  >*Kopier importId*  

* Sjekk status på import job
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "WAITING_FOR_FILE"
    .
    .
  }
  ```
* Importer fil
  >**GET /imports/{importId}/archive**  
  >ImportId:*importId fra tidligere steg*  
  >file: *choose file* - velg zip filen du lastet ned fra mlds i første steg  
  >*Execute*  
* Vent 5ish min og ta en ny sjekk av status på import job som over. Hvis status er "RUNNING", vent litt til
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "COMPLETED"
    .
    .
  }
  ```
* Norsk ekstensjon
* Diagnoser
* Legemidler
* ICN

* *!!!-- prosedyre stop --!!!*

* Importere ICPC2 til prod fra [backend](https://app-fsre-backend-prod-noe-1.azurewebsites.net/index.html)
* ##### !!!!! NB vi måtte bruke ehelse.no endepunkter denne gangen da det sannsynligvis mangler noe nginx config på snowstorm serverene
* Trykk på **Authorize** knappen oppe til høyre (Trengs bare første gang man bruker backend)
* Kjør sync
  >**POST** /api/RefsetTools/sync
  >Request body:
  ```
  {
    "userName": "username",
    "password": "password",
    "sourceBaseUri": "https://dailybuild.terminologi.helsesdirektoratet.no/snowstorm/snomed-ct",
    "sourceBranch": "MAIN/SNOMEDCT-NO/REFSETS",
    "targetBaseUri": "https://snowstorm.terminologi.helsedirektoratet.no/snowstorm/snomed-ct",
    "targetBranch": "MAIN/SNOMEDCT-NO",
    "refset": {
      "sourceRefsetId": "68101000202102",
      "sourceModuleId": "51000202101",
      "targetRefsetId": "68101000202102",
      "targetModuleId": "51000202101",
      "includeMembers": false,
      "deleteFromTarget": false,
      "deleteDiffsFromTarget": false,
      "addToTarget": true,
      "addWithNewMemberId": false
    }
  }
  ```
* Gå tilbake til [prod](https://snowstorm.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/) og lag en ny versjon av SNOMEDCT-NO
  >**POST /codesystems/{shortName}/versions  
  >shortname: SNOMEDCT-NO  
  >Request body:  
  ```
  {
    "effectiveDate": 20YYMM15,
    "description": "[Month] release, norsk ekstensjon",
    "internalRelease": false
  }
  ```
  >*Execute*  
---

### Importere norsk ekstensjon og derivater i [dailybuild](https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/)

* Rollback den tidligere importerte releasen
* Finn headTimestamp som man skal rulle tilbake til
  >**GET** /branches/{branch}  
  >branch: MAIN/SNOMEDCT-NO
  >*Execute*  
  >Response body:
  ```
  {
    .
    .
    "headTimestamp": 1722954366627,
    .
    .
  }
  ```
  >*Kopier headTimestamp*  
* Utfør rollback-commit
  >**POST** /admin/{branch}/actions/rollback-commit
  >branch: MAIN/SNOMEDCT-NO  
  >commitHeadTime: *headTimestamp fra over*
  >*Execute*  
* Sjekk at NO versjonen som ble importert før export av derivater er borte fra branchen
  >**GET** branches  
  >*Execute*  

* Start med norsk ekstensjon, gjenta deretter prosedyren under med rf2 pakkene lastet ned fra Azure (Diagnoser, Legemidler og ICN):

* *!!!-- prosedyre start --!!!*

* Lag en import job
  >**POST /imports**
  >Request body:
  ```
  {
    "type": "SNAPSHOT", 
    "branchPath": "MAIN/SNOMEDCT-NO", 
    "createCodeSystemVersion": "false"  
  }
  ```
  >*Execute*  
  >Response body:  
  ```
  {
    .
    .
    location: https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/imports/[LANG importId TALLSTRENG HER]
    .
    .
  }
  ```
  >*Kopier importId*  

* Sjekk status på import job
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "WAITING_FOR_FILE"
    .
    .
  }
  ```
* Importer fil
  >**GET /imports/{importId}/archive**  
  >ImportId:*importId fra tidligere steg*  
  >file: *choose file* - velg zip filen du lastet ned fra mlds i første steg  
  >*Execute*  
* Vent 5ish min og ta en ny sjekk av status på import job som over. Hvis status er "RUNNING", vent litt til
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "COMPLETED"
    .
    .
  }
  ```
* Norsk ekstensjon
* Diagnoser
* Legemidler
* ICN

* *!!!-- prosedyre stop --!!!*

* Importere ICPC2 til dailybuild fra [backend](https://app-fsre-backend-prod-noe-1.azurewebsites.net/index.html)
* ##### !!!!! NB vi måtte bruke ehelse.no endepunkter denne gangen da det sannsynligvis mangler noe nginx config på snowstorm serverene
* Utfør sync
  >**POST /api/RefsetTools/sync
  >Request body:
  ```
  {
    "userName": "username",
    "password": "password",
    "sourceBaseUri": "https://dailybuild.terminologi.helsesdirektoratet.no/snowstorm/snomed-ct",
    "sourceBranch": "MAIN/SNOMEDCT-NO/REFSETS",
    "targetBaseUri": "https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct",
    "targetBranch": "MAIN/SNOMEDCT-NO",
    "refset": {
      "sourceRefsetId": "68101000202102",
      "sourceModuleId": "51000202101",
      "targetRefsetId": "68101000202102",
      "targetModuleId": "51000202101",
      "includeMembers": false,
      "deleteFromTarget": false,
      "deleteDiffsFromTarget": false,
      "addToTarget": true,
      "addWithNewMemberId": false
    }
  }
  ```
* Gå tilbake til [dailybuild](https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/) og lag en ny versjon av SNOMEDCT-NO
  >**POST /codesystems/{shortName}/versions  
  >shortname: SNOMEDCT-NO  
  >Request body:  
  ```
  {
    "effectiveDate": 20YYMM15,
    "description": "[Month] release, norsk ekstensjon",
    "internalRelease": false
  }
  ```
  >*Execute*  
---

### Importere norsk ekstensjon og derivater for [demo](https://demo.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/)
* Rollback den tidligere importerte releasen
* Finn headTimestamp som man skal rulle tilbake til
  >**GET** /branches/{branch}  
  >branch: MAIN/SNOMEDCT-NO
  >*Execute*  
  >Response body:
  ```
  {
    .
    .
    "headTimestamp": 1722954366627,
    .
    .
  }
  ```
  >*Kopier headTimestamp*  
* Utfør rollback-commit
  >**POST** /admin/{branch}/actions/rollback-commit
  >branch: MAIN/SNOMEDCT-NO  
  >*Execute*  
* Sjekk at NO versjonen som ble importert før export av derivater er borte fra branchen
  >**GET** branches  
  >*Execute*  

* Start med norsk ekstensjon, gjenta deretter prosedyren under med rf2 pakkene lastet ned fra Azure (Diagnoser, Legemidler og ICN):

* *!!!-- prosedyre start --!!!*

* Lag en import job
  >**POST /imports**
  >Request body:
  ```
  {
    "type": "SNAPSHOT", 
    "branchPath": "MAIN/SNOMEDCT-NO", 
    "createCodeSystemVersion": "false"  
  }
  ```
  >*Execute*  
  >Response body:  
  ```
  {
    .
    .
    location: https://demo.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/imports/[LANG importId TALLSTRENG HER]
    .
    .
  }
  ```
  >*Kopier importId*  

* Sjekk status på import job
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "WAITING_FOR_FILE"
    .
    .
  }
  ```
* Importer fil
  >**GET /imports/{importId}/archive**  
  >ImportId:*importId fra tidligere steg*  
  >file: *choose file* - velg zip filen du lastet ned fra mlds i første steg  
  >*Execute*  
* Vent 5ish min og ta en ny sjekk av status på import job som over. Hvis status er "RUNNING", vent litt til
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "COMPLETED"
    .
    .
  }
  ```
* Norsk ekstensjon
* Diagnoser
* Legemidler
* ICN

* *!!!-- prosedyre stop --!!!*

* ##### !!!!! NB vi måtte bruke ehelse.no endepunkter denne gangen da det sannsynligvis mangler noe nginx config på snowstorm serverene
* Importere ICPC2 til dailybuild fra [backend](https://app-fsre-backend-prod-noe-1.azurewebsites.net/index.html)
* Kjør sync
  >**POST /api/RefsetTools/sync
  >Request body:
  ```
  {
    "userName": "username",
    "password": "password",
    "sourceBaseUri": "https://dailybuild.terminologi.helsesdirektoratet.no/snowstorm/snomed-ct",
    "sourceBranch": "MAIN/SNOMEDCT-NO/REFSETS",
    "targetBaseUri": "https://demo.terminologi.helsedirektoratet.no/snowstorm/snomed-ct",
    "targetBranch": "MAIN/SNOMEDCT-NO",
    "refset": {
      "sourceRefsetId": "68101000202102",
      "sourceModuleId": "51000202101",
      "targetRefsetId": "68101000202102",
      "targetModuleId": "51000202101",
      "includeMembers": false,
      "deleteFromTarget": false,
      "deleteDiffsFromTarget": false,
      "addToTarget": true,
      "addWithNewMemberId": false
    }
  }
  ```
* Gå tilbake til [demo](https://demo.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/) og lag en ny versjon av SNOMEDCT-NO
  >**POST /codesystems/{shortName}/versions  
  >shortname: SNOMEDCT-NO  
  >Request body:  
  ```
  {
    "effectiveDate": 20YYMM15,
    "description": "[Month] release, norsk ekstensjon",
    "internalRelease": false
  }
  ```
  >*Execute*  
---


### Importere norsk ekstensjon og derivater for [slv](https://slv.terminologi.helsedirektoratet.no/swagger-ui/index.html#/)
* Verifiser at SNOMEDCT-NO har riktig avhengighet til den internasjonale utgaven
  >**GET /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
  ```
  {
    .
    .
    "dependantVersionEffectiveTime": YYYYMMDD,
    "dailyBuildAvailable": false,
    .
    .
  }
  ```
* Start med norsk ekstensjon, gjenta deretter prosedyren under med rf2 pakkene lastet ned fra Azure (Diagnoser, Legemidler og ICN):

* *!!!-- prosedyre start --!!!*

* Lag en import job
##### !!! 2024-09-15 createCodeSystemVersion: true da det kun er norsk ekstensjon som skal lastes inn
  >**POST /imports**
  Request body:
  ```
  {
    "type": "SNAPSHOT", 
    "branchPath": "MAIN/SNOMEDCT-NO", 
    "createCodeSystemVersion": "false"  
  }
  ```
  >*Execute*  
  >Response body:  
  ```
  {
    .
    .
    location: https://test.slv.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/imports/[LANG importId TALLSTRENG HER]
    .
    .
  }
  ```
  >*Kopier importId*  

* Sjekk status på import job
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "WAITING_FOR_FILE"
    .
    .
  }
  ```
* Importer fil
  >**GET /imports/{importId}/archive**  
  >ImportId:*importId fra tidligere steg*  
  >file: *choose file* - velg zip filen du lastet ned fra mlds i første steg  
  >*Execute*  
* Vent 5ish min og ta en ny sjekk av status på import job som over. Hvis status er "RUNNING", vent litt til
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "COMPLETED"
    .
    .
  }
  ```
##### 2024-009-15 Kun norsk ekstensjon, IKKE derivater
* Norsk ekstensjon
* Diagnoser 
* Legemidler
* ICN

* *!!!-- prosedyre stop --!!!*

##### 2024-09-15 derivater er ikke importert og da kjører vi heller ikke ICPC2 sync
* Importere ICPC2 til prod fra [backend](https://app-fsre-backend-prod-noe-1.azurewebsites.net/index.html)
* Trykk på **Authorize** knappen oppe til høyre
  >**POST** /api/RefsetTools/sync
  >Request body:
  ```
  {
    "userName": "username",
    "password": "password",
    "sourceBaseUri": "https://dailybuild.terminologi.helsesdirektoratet.no/snowstorm/snomed-ct",
    "sourceBranch": "MAIN/SNOMEDCT-NO/REFSETS",
    "targetBaseUri": "https://slv.terminologi.helsedirektoratet.no/snowstorm/snomed-ct",
    "targetBranch": "MAIN/SNOMEDCT-NO",
    "refset": {
      "sourceRefsetId": "68101000202102",
      "sourceModuleId": "51000202101",
      "targetRefsetId": "68101000202102",
      "targetModuleId": "51000202101",
      "includeMembers": false,
      "deleteFromTarget": false,
      "deleteDiffsFromTarget": false,
      "addToTarget": true,
      "addWithNewMemberId": false
    }
  }
  ```
##### 2024-09-15 ikke nødvendig da importen ble kjørt med createCodeSystemVersion: true
* Gå tilbake til [slv](https://slv.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/) og lag en ny versjon av SNOMEDCT-NO
  >**POST /codesystems/{shortName}/versions  
  >shortname: SNOMEDCT-NO  
  >Request body:  
  ```
  {
    "effectiveDate": 20YYMM15,
    "description": "[Month] release, norsk ekstensjon",
    "internalRelease": false
  }
  ```
  >*Execute*  

---

### Importere norsk ekstensjon og derivater for [slv-test](https://test.slv.terminologi.helsedirektoratet.no/swagger-ui/index.html#/)
* Verifiser at SNOMEDCT-NO har riktig avhengighet til den internasjonale utgaven
  >**GET /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
  ```
  {
    .
    .
    "dependantVersionEffectiveTime": YYYYMMDD,
    "dailyBuildAvailable": false,
    .
    .
  }
  ```
* Start med norsk ekstensjon, gjenta deretter prosedyren under med rf2 pakkene lastet ned fra Azure (Diagnoser, Legemidler og ICN):

* *!!!-- prosedyre start --!!!*

##### !!! 2024-09-15 createCodeSystemVersion: true da det kun er norsk ekstensjon som skal lastes inn
* Lag en import job
  >**POST /imports**
  Request body:
  ```
  {
    "type": "SNAPSHOT", 
    "branchPath": "MAIN/SNOMEDCT-NO", 
    "createCodeSystemVersion": "false"  
  }
  ```
  >*Execute*  
  >Response body:  
  ```
  {
    .
    .
    location: https://test.slv.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/imports/[LANG importId TALLSTRENG HER]
    .
    .
  }
  ```
  >*Kopier importId*  

* Sjekk status på import job
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "WAITING_FOR_FILE"
    .
    .
  }
  ```
* Importer fil
  >**GET /imports/{importId}/archive**  
  >ImportId:*importId fra tidligere steg*  
  >file: *choose file* - velg zip filen du lastet ned fra mlds i første steg  
  >*Execute*  
* Vent 5ish min og ta en ny sjekk av status på import job som over. Hvis status er "RUNNING", vent litt til
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
    "status": "COMPLETED"
    .
    .
  }
  ```
##### 2024-009-15 Kun norsk ekstensjon, IKKE derivater
* Norsk ekstensjon
* Diagnoser
* Legemidler
* ICN

* *!!!-- prosedyre stop --!!!*

##### 2024-09-15 derivater er ikke importert og da kjører vi heller ikke ICPC2 sync
* Importere ICPC2 til prod fra [backend](https://app-fsre-backend-prod-noe-1.azurewebsites.net/index.html)
* Trykk på **Authorize** knappen oppe til høyre
  >**POST** /api/RefsetTools/sync
  >Request body:
  ```
  {
    "userName": "username",
    "password": "password",
    "sourceBaseUri": "https://dailybuild.terminologi.helsesdirektoratet.no/snowstorm/snomed-ct",
    "sourceBranch": "MAIN/SNOMEDCT-NO/REFSETS",
    "targetBaseUri": "https://test.slv.terminologi.helsedirektoratet.no/snowstorm/snomed-ct",
    "targetBranch": "MAIN/SNOMEDCT-NO",
    "refset": {
      "sourceRefsetId": "68101000202102",
      "sourceModuleId": "51000202101",
      "targetRefsetId": "68101000202102",
      "targetModuleId": "51000202101",
      "includeMembers": false,
      "deleteFromTarget": false,
      "deleteDiffsFromTarget": false,
      "addToTarget": true,
      "addWithNewMemberId": false
    }
  }
  ```
##### 2024-09-15 ikke nødvendig da importen ble kjørt med createCodeSystemVersion: true
* Gå tilbake til [test-slv](https://test.slv.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/) og lag en ny versjon av SNOMEDCT-NO
  >**POST /codesystems/{shortName}/versions  
  >shortname: SNOMEDCT-NO  
  >Request body:  
  ```
  {
    "effectiveDate": 20YYMM15,
    "description": "[Month] release, norsk ekstensjon",
    "internalRelease": false
  }
  ```
  >*Execute*  
---
---
### Delete and create MAIN/SNOMEDCT-NO/REFSETS på prod, demo og dailyBuild
* !!!-- prosedyre start --!!! 
* Delete REFSETS
>**DELETE** /admin/{branch}/actions/hard-delete  
>branch: MAIN/SNOMEDCT-NO/REFSETS
>*Execute*  
* Create REFSETS
>**POST /branches**  
>Request body  
```
{
  "parent": "MAIN/SNOMEDCT-NO",
  "name": "REFSETS"
}
```
* !!!-- prosedyre stop --!!!
* prod
* dailyBuild
* demo
### Enable dailyBuild 
* dailyBuild
##### !!! 2024-09-15 ikke demo da den var av i utgangspunktet
* demo
---
---
---
