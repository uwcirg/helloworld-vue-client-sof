<template>
  <div id="interventionElement" v-if="ready">
    <h2 class="font-weight-bold text-h5 mb-4 mt-2" v-text="title"></h2>
    <v-card elevation="0">
        <h4>Patient Info</h4>
        <table>
          <tr>
            <td class="mr-2">Name</td>
            <td v-text="summary.Patient.Name"></td>
          </tr>
           <tr>
            <td class="mr-2">Gender</td>
            <td v-text="summary.Patient.Gender"></td>
          </tr>
           <tr>
            <td class="mr-2">Age</td>
            <td v-text="summary.Patient.Age"></td>
          </tr>
        </table>
    </v-card>
  </div>
</template>

<script>
  export default {
    props: {
      summary: [Array, Object],
      title: String
    },
    watch: {
      'summary': function() {
        this.ready = true;
      }
    },
    data() {
      // Indicate we are not yet ready for the component to render
      return {
        ready: false,
        tab: 0
      };
    },
    methods: {
      hasResponses () {
        return this.summary;
      }
    }
  };
</script>
<style>/*component specific css here*/</style>
