<template>
  <div>
    <!-- search input -->
    <div class="ipc-search mt-6 mt-sm-0">
      <v-text-field
        v-model="search"
        append-icon="mdi-magnify"
        label="Search"
        single-line
        hide-details
        class="pb-5"
      ></v-text-field>
    </div>
    <!-- table alert -->
    <v-alert v-if="showAlert" :type="alertType" tile dense>{{alertMessage}}</v-alert>
    <!-- table header -->
    <v-data-table
      :headers="headers"
      :items="submissions"
      :items-per-page="10"
      :search="search"
      :loading="loading"
      loading-text="Loading... Please wait"
      item-key="confirmationId"
      class="ipc-table"
    >
      <template v-slot:item.download="{ item }">
        <GeneratePdfButton :ipcPlanId="item.ipcPlanId">
          <v-btn text small color="textLink"><v-icon class="mr-1">picture_as_pdf</v-icon> PDF</v-btn>
        </GeneratePdfButton>
      </template>

      <template v-slot:item.details="{ item }">
        <router-link :to="{ name: 'Submission', params: { ipcPlanId: item.ipcPlanId } }">
          <v-btn text small color="textLink"><v-icon class="mr-1">remove_red_eye</v-icon> VIEW</v-btn>
        </router-link>
      </template>
    </v-data-table>
  </div>
</template>

<script>
import GeneratePdfButton from '@/components/common/GeneratePdfButton.vue';
import ipcService from '@/services/ipcService';

export default {
  name: 'SubmissionsTable',
  components: {
    GeneratePdfButton
  },
  computed: {
    responsiveCell () {
      return (this.$vuetify.breakpoint.name == 'xs') ? 'v-data-table__mobile-table-row' : '';
    }
  },
  data() {
    return {
      // vuetify data table
      search: '',
      headers: [
        { text: 'Submitted', value: 'created' },
        { text: 'Business Name', align: 'start', value: 'name' },
        { text: 'Confirmation ID', align: 'start', value: 'confirmationId' },
        { text: 'Download', value: 'download', sortable: false },
        { text: 'Details', value: 'details', sortable: false }
      ],
      submissions: [],
      loading: true,
      showAlert: false,
      alertType: null,
      alertMessage: ''
    };
  },
  methods: {
    formatDate(date){
      return (date) ? new Date(date).toLocaleString() : 'N/A';
    },
    // get table data from frontend service layer
    getData() {
      ipcService
        .getAllIPCMetaData()
        .then(response => {
          const data = response.data;
          const submissions = Object.keys(data).map(k => {
            let submission = data[k];
            return {
              ipcPlanId: submission.ipcPlan.ipcPlanId,
              pdf: submission.ipcPlan.ipcPlanId,
              name: submission.business.name,
              created: this.formatDate(submission.ipcPlan.createdAt),
              confirmationId: submission.confirmationId,
            };
          });
          if (!submissions.length) {
            this.showTableAlert('info', 'No Submissions found');
          }
          this.submissions = submissions;
        })
        .catch(() => {
          this.showTableAlert('error', 'No response from server');
        });
    },
    showTableAlert(typ, msg) {
      this.showAlert = true;
      this.alertType = typ;
      this.alertMessage = msg;
      this.loading = false;
    },
  },
  mounted() {
    this.getData();
  },
  watch: {
    // hide data table progress bar when submissions have been returned from backend
    submissions() {
      this.loading = false;
    }
  }
};
</script>

<style scoped>
.ipc-search {
  width: 100%;
}
@media (min-width: 600px) {
  .ipc-search {
    max-width: 20em;
    float: right;
  }
}
@media (max-width: 599px) {
  .ipc-search {
    padding-left: 16px;
    padding-right: 16px;
  }
}

.ipc-table {
  clear: both;
}
/* Want to use scss but the world hates me */
.ipc-table >>> tbody tr:nth-of-type(odd) {
  background-color: #f5f5f5;
}
.ipc-table >>> thead tr th {
  font-weight: normal;
  color: #003366 !important;
  font-size: 1.1em;
}
</style>
