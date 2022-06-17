<template>
  <v-app>
    <Header :patient="patient"></Header>
    <v-main>
      <v-container fluid>
        <!-- loading indicator -->
        <v-progress-circular
        :value="160"
        indeterminate
        color="primary"
        v-if="!error && !ready"
        ></v-progress-circular>
        <div id="app">
          <!-- body -->
          <Summary
            :title="title"
            :summary="summary"
          />
        </div>
        <!-- error message -->
        <v-alert
          color="error"
          dark
          v-if="error">
          Error loading content.  See console for detail.
          <div v-html="error"></div>
        </v-alert>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import Worker from '../node_modules/cql-worker/src/cql.worker.js'; // https://github.com/webpack-contrib/worker-loader
import { initialzieCqlWorker } from 'cql-worker';
import FHIR from 'fhirclient';
import { getFHIRResourcePaths, getExpressionLogicLib, queryPatientIdKey} from './util/util.js';
import Header from './components/Header.vue';
import Summary from './components/Summary.vue';
import './style/app.scss';

// Define a web worker for evaluating CQL expressions
const cqlWorker = new Worker();
// Initialize the cql-worker
let [setupExecution, sendPatientBundle, evaluateExpression] = initialzieCqlWorker(cqlWorker);

export default {
  name: 'App',
  components: {
    Summary,
    Header,
  },
  data: function() {
    return {
      client: null,
      title: 'Hello World App',
      patientBundle: {
        resourceType: 'Bundle',
        id: 'resource-bundle',
        type: 'collection',
        entry: []
      },
      summary: {},
      patient: null,
      error: "",
      ready: false,
    }
  },
  async mounted() {
    //authorized FHIR client
    this.setAuthClient().then((result) => {
      this.client = result;
      if (this.error) return; // auth error, cannot continue
      //set patient info
      this.setPatient().then((pt) => {
        if (this.error) return;
        if (!pt || !pt.id) {
          this.error = 'No valid patient.';
          return 
        }
        this.patient = pt;
        this.patientId = pt.id;
        this.patientBundle.entry.unshift({resource: this.patient});

        //get FHIR resources for the patient
        this.getFhirResources().then(() => {
        
          //add FHIR resources for the patient to the summary object
          this.summary['resources'] = this.patientBundle;
          console.log('summary with resources ', this.summary);

          //get CQL expression lib
          getExpressionLogicLib('Summary').then(data => {

            // Load CQL ELM JSON, and value set cache which contains evaluated expressions for use by app
            const [elmJson, valueSetJson, namedExpression] = data;

            // Send the cqlWorker an initial message containing the ELM JSON representation of the CQL expressions
            setupExecution(elmJson, valueSetJson);
            // Send patient info to CQL worker to process
            sendPatientBundle(this.patientBundle);
       
            setTimeout(() => {
              //named expression here is Summary, look in /src/cql/source/ExpressionLogicLibrary.cql and see how that is defined
              evaluateExpression(namedExpression).then(result => {
                console.log("CQL expression result ", result)
                if (result) {
                  this.summary = {...this.summary, ...result};
                  console.log('summary with CQL result added ', this.summary);
                }
              }).catch( e => {
                this.error = e;
                console.log("CQL Expression evaluation error ", e);
              });
              this.ready = true;
            }, 50);
          });
            
        }).catch(e => {
          this.error = e;
          console.log("FHIR resources error ", e);
        })

      }).catch(e => {
        this.error = e;
        console.log("Patient resource error ", e);
      });
    }).catch(e => {
      console.log("Auth Error ", e);
      this.error = e;
      this.ready = true;
    });
  },
  methods: {
    async setAuthClient() {
       // Wait for authorization
      return await FHIR.oauth2.ready();
    },
    async setPatient() {
      //this is a workaround for when patient id is not embedded within the JWT token
      let queryPatientId = sessionStorage.getItem(queryPatientIdKey);
      if (queryPatientId) {
        console.log("Using stored patient id ", queryPatientId);
        return this.client.request('/Patient/'+queryPatientId);
      }
       // Get the Patient resource
      return await this.client.patient.read().then((pt) => {
        return pt;
      });
    },
    async getFhirResources() {
      const resources = getFHIRResourcePaths(this.patientId);
      const requests = resources.map(resource => this.client.request(resource));
      return Promise.all(requests).then(results => {
        results.forEach(result => {
          if (!result) return true;
          if (result.resourceType == 'Bundle' && result.entry) {
            result.entry.forEach(o => {
              if (o && o.resource) this.patientBundle.entry.push({resource: o.resource});
            });
          } else if (Array.isArray(result)) {
            result.forEach(o => {
              if (o.resourceType) this.patientBundle.entry.push({resource: o});
            });
          } else {
            this.patientBundle.entry.push({resource: result});
            if (result.resourceType.toLowerCase() === "patient") this.patient = result;
          }
        });
        console.log("FHIR resource bundles ", this.patientBundle.entry);
      });
    }
  }
}
</script>
