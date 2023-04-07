<template>
  <v-data-table
    :options.sync="options"
    :server-items-length="donor_count"
    :loading="loading"
    :headers="headers"
    :items="donor_list.data"
    :items-per-page="10"
    class="elevation-1 donor-list-table"
  >
    <template v-slot:top>
      <v-text-field
        v-model="search"
        label="Search by name or email"
        class="mx-4 pt-4"
        outlined
        clearable
        prepend-inner-icon="mdi-magnify"
      ></v-text-field>
      <v-alert v-if="error" type="error" class="mx-4 pt-4">{{ error }}</v-alert>
    </template>
    <template v-slot:[`item.first_donation`]="{ item }">
      {{ new Date(item.first_donation).toLocaleDateString() }}
    </template>
    <template v-slot:progress>
      <v-progress-linear color="#00754A" indeterminate></v-progress-linear>
    </template>
  </v-data-table>
</template>
<style>
@import '../styles/DonorListTable.scss';
</style>
<script>
import { debounce } from 'lodash';
export default {
  name: 'DonationTable',
  data() {
    return {
      headers: [
        { text: 'NAME', value: 'full_name' },
        { text: 'EMAIL', value: 'email' },
        {
          text: 'TOTAL DONATIONS',
          value: 'total_donations',
        },
        { text: 'FIRST DONATION', value: 'first_donation' },
      ],
      donor_count: 0,
      search: '',
      donor_list: [],
      loading: false,
      available_sorting: [],
      error: '',
      options: {
        page: 1,
        itemsPerPage: 10,
        sortBy: ['full_name'],
      },
    };
  },
  watch: {
    options: {
      handler() {
        this.getDonorList();
      },
      deep: true,
    },
    search: {
      handler() {
        this.getDonorList();
      },
      deep: true,
    },
  },
  methods: {
    getDonorList: debounce(async function () {
      const { page, itemsPerPage, sortBy, sortDesc } = this.options;
      this.error = '';
      this.loading = true;
      try {
        const response = await fetch(
          `https://interview.ribbon.giving/api/donors?page=${page}&itemsPerPage=${itemsPerPage}&search=${
            this.search
          }&sort_by=${sortBy[0] ? sortBy[0] : 'full_name'}&sort_direction=${sortDesc[0] ? 'desc' : 'asc'}`
        );
        const resdata = await response.json();
        this.donor_list = resdata;
        this.donor_count = resdata.meta.total;
        this.available_sorting = resdata.filtering.available_sorting;
        this.loading = false;
      } catch (err) {
        this.error = err.message || 'There was an error fetching your data, please try again.';
        this.loading = false;
      }
    }, 500),
  },
};
</script>
