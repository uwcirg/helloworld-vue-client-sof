<template>
   <v-app-bar
        dark
        color="primary"
        elevation=2
        app>
        <v-toolbar-title class="text-h5">Patient Viewer</v-toolbar-title>
        <div class="ml-6">
            <div class="text-subtitle-1 white--text" v-if="hasPatientName()" v-text="getPatientName()"></div>
            <div class="text-subtitle-1 white--text" v-text="getPatientDob()"></div>
        </div>
    </v-app-bar>
</template>

<script>
  export default {
    props: {
      patient: Object
    },
    methods: {
      hasPatientName () {
        return this.patient && this.patient.name && this.patient.name.length;
      },
      getPatientName () {
        if (!this.hasPatientName()) return '';
        return [this.patient.name[0].family, this.patient.name[0].given[0]].join(', ');
      },
      getPatientDob () {
        if (!this.patient || !this.patient.birthDate) return '';
        return this.patient.birthDate;
      }
    }
  };
</script>
