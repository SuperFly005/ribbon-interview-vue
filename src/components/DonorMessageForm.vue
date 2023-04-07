<template>
  <v-sheet width="400" class="mx-auto">
    <v-form class="mx-4 my-4" v-model="valid" validate-on="submit" @submit.prevent="submit">
      <v-textarea v-model="message" :rules="messageValidator" label="Message" validate-on-blur></v-textarea>
      <v-autocomplete
        v-model="email"
        v-on:focus="getDonorEmails"
        :loading="loading"
        :items="donorEmails"
        :search-input.sync="emailSearchQuery"
        :rules="emailValidator"
        label="Email"
        class="my-2 py-2"
        flat
        cache-items
        hide-details
        hide-no-data
        validate-on-blur
      ></v-autocomplete>
      <v-text-field v-model="donor_id" label="Donor Id" readonly></v-text-field>
      <v-btn type="submit" block class="mt-2 mb-4" :loading="submitting">SEND</v-btn>
      <v-alert v-if="error" type="error" class="pt-4">{{ error }}</v-alert>
      <v-alert v-if="success" type="success" class="pt-4">Message Sent.</v-alert>
    </v-form>
  </v-sheet>
</template>

<script>
export default {
  name: 'DonorMessageForm',

  data() {
    return {
      donorEmails: [],
      emailToDonorId: {},
      emailSearchQuery: null,
      message: '',
      email: null,
      donor_id: '',
      valid: false,
      error: '',
      success: false,
      submitting: false,
      loading: false,
      emailValidator: [
        (value) => {
          if (value) return true;
          this.valid = false;
          return 'e-mail is required.';
        },
      ],
      messageValidator: [
        (value) => {
          if (value?.length > 15) return true;
          this.valid = false;
          return 'Message must be at least 15 characters.';
        },
      ],
    };
  },
  watch: {
    emailSearchQuery: {
      handler() {
        this.getDonorEmails();
      },
      deep: true,
    },
    email: {
      handler() {
        this.donor_id = this.emailToDonorId[this.email];
      },
    },
  },
  methods: {
    async getDonorEmails() {
      this.error = '';
      this.loading = true;
      try {
        const response = await fetch(
          `https://interview.ribbon.giving/api/donors?search=${this.emailSearchQuery ? this.emailSearchQuery : ''}`
        );
        const resData = await response.json();
        this.donorEmails = resData.data.map((d) => d.email);
        this.emailToDonorId = resData.data.reduce((acc, { email, id }) => {
          acc[email] = id;
          return acc;
        }, {});
        this.loading = false;
      } catch (err) {
        this.error = err.message || 'An error occurred while fetching the email data. Please try again.';
        this.loading = false;
      }
    },
    async submit() {
      this.success = false;
      this.error = '';
      if (this.valid) {
        try {
          this.submitting = true;
          const response = await fetch(`https://interview.ribbon.giving/api/donors/${this.donor_id}/send-message`, {
            method: 'POST',
            headers: {
              'Content-type': 'application/json',
            },
            body: JSON.stringify({
              email: this.email,
              donor_id: this.donor_id,
              message: this.message,
            }),
          });
          const resData = await response.json();
          this.success = resData.Success;
          if (this.success) {
            this.email = null;
            this.donor_id = '';
            this.message = '';
          } else {
            this.error = resData.message || 'An error occurred while sending the message. Please try again.';
          }
          this.submitting = false;
        } catch (err) {
          this.error = err.message || 'An error occurred while sending the message. Please try again.';
          this.submitting = false;
        }
      }
    },
  },
};
</script>
