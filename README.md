# Hello World Smart on FHIR app
This is modeled after the [ASBI Screening App](https://github.com/asbi-cds-tools/asbi-screening-app/). The app can be used in conjuction with the [SMART<sup>&reg;</sup> app launch framework](http://hl7.org/fhir/smart-app-launch/index.html).

### Clinical Quality Language (CQL)
[CQL](https://cql.hl7.org/) is a domain-specific programming language focused on clinical quality applications, including CDS as well as electronic clinical quality measures (eCQMs). Logical expressions written in CQL are human-readable but can also be compiled to a machine-friendly format to facilitate implementation. This application executes CQL logic to provide patient customized behavior. Machine-friendly versions of the CQL are embedded in this app; For more information about CQL see [here](https://cql.hl7.org/); For more information about how to compile CQL code into machine-readable format, see [here](https://github.com/cqframework/clinical_quality_language).

## Underlying Technologies

### Vue.js
[Vue](https://vuejs.org/) is a JavaScript front-end framework for building user interfaces. The application was built using the [`vue create` command](https://cli.vuejs.org/guide/creating-a-project.html#vue-create) from the Vue command line interface (CLI).

### Vuetify
[Vuetify](https://vuetifyjs.com/en/) is a Vue UI library with Material Design themed UI components that can be used for the application.

### CQL Execution Engine
All CQL calculations are executed using the [CQL Execution Engine](https://github.com/cqframework/cql-execution), an open source library that implements the CQL standard.

### Web Workers
All CQL calculations are executed within the context of a [Web Worker](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers), thereby offloading them to a separate thread. This greatly improves the responsiveness of the application.

## Usage
A number of options are available for local usage to support testing with synthetic data.

### Setup
This project manages dependencies using the [Yarn package manager](https://yarnpkg.com/) in the [Node environment](https://nodejs.dev/) (Node version <= 16 is recommended for this application). Make sure to have both Yarn and Node installed before proceeding. The dependencies for the application can be installed locally by typing `yarn` at the command line. A local version of the app can be launched by typing `yarn serve` at the command line. A copy suitable for distribution can be built using the `yarn build` command.

### Download Value Sets from VSAC
The value set content used by the CQL is cached in a file named valueset-db.json, which has been checked into this project in an empty state. In order for the CDS to operate as intended, implementers must populate valueset-db.json with the value sets which have been published on the [Value Set Authority Center (VSAC)](https://vsac.nlm.nih.gov/). In order to access VSAC, you must sign up for a [UMLS Terminology Services account](https://uts.nlm.nih.gov//license.html).

Once a UMLS Terminology Services account has been obtained, the valueset-db.json file can be updated by running the following:

1. Run `node src/util/updateValueSetDB.js UMLS_API_KEY` _(replacing UMLS\_API\_KEY with your actual UMLS API key)_

To get you UMLS API Key:

1. Sign into your UMLS account at [https://uts.nlm.nih.gov/uts.html](https://uts.nlm.nih.gov/uts.html)
2. Click 'My Profile' in the orange banner at the top of the screen
3. Your API key should be listed below your username in the table
4. If no API key is listed:
   1. Click ‘Edit Profile’
   2. Select the ‘Generate new API Key’ checkbox
   3. Click ‘Save Profile’
   4. Your new API key should now be listed.

### Configuration
Parameters for the app are stored in [environmental variables](http://man7.org/linux/man-pages/man7/environ.7.html) that are stored in an `.env` file (`cp default.env .env` to allow environment variables to be read by the application). The [dotenv package](https://www.npmjs.com/package/dotenv) is used to store the default variable values, which can be overwritten by defining a more specific env (e.g., `.env.local`) file or by setting the variables in the deployment system. For more information, see the [Vue documentation](https://cli.vuejs.org/guide/mode-and-env.html#environment-variables).

#### Parameters

| Parameter | Description | Allowed Values |
| --- | --- | --- |
| `VUE_APP_FHIR_RESOURCES` | Define the FHIR resource(s) to load for the patient | `Condition,Procedure,Observation,Questionnaire,QuestionnaireResponse` |
| `VUE_APP_FHIR_OBSERVATION_CATEGORIES` | Define what categor(ies) of FHIR observations to load for the patient | `social-history,vital-signs,imaging,laboratory,procedure,survey,exam,therapy,activity` |
| `VUE_APP_AUTH_SCOPES` | For allowing the app to specify the delegation of a specific set of access rights via launch context. see [App Launch: Scopes and Launch Context](https://build.fhir.org/ig/HL7/smart-app-launch/scopes-and-launch-context.html) | `profile roles email patient/*.read openid fhirUser patient/QuestionnaireResponse.write` |
| `VUE_APP_SYSTEM_TYPE` | Define system type | `development`, `demo`, `staging`, `production`
### Using with Public SMART Sandbox
A public [SMART<sup>&reg;</sup> App Launcher](https://launch.smarthealthit.org/index.html) is available for sandbox tesing of SMART on FHIR apps with synthetic data.

#### Launch for SMART App Launcher
Navigate to the public SMART<sup>&reg;</sup> App Launcher and choose the "Provider EHR Launch" Launch Type. Uncheck "Simulate launch within the EHR user interface".  Leave all other options unselected. Paste the URL to where `launch.html`, e.g. `http://localhost:8080/launch.html`, is being served from into the "App Launch URL" box at the bottom of the SMART<sup>&reg;</sup> App Launcher page. Select "Launch App!" which will bring up a patient selector widget before the *ASBI Intervention App* is launched.

