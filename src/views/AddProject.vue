<template>
  <v-container>
    <v-card
      class="mx-auto my-12 pt-3"
      elevation="2"
      max-width="450"
    >
      <v-card-text>
        <h1>Create Project</h1>
        <form class="mt-4">
          <v-text-field
            v-model="title"
            label="Title"
            outlined
            dense
            clearable
          ></v-text-field>

          <v-textarea
            solo
            class="mb-0"
            rows="4"
            label="Description"
            v-model="description"
          ></v-textarea>

          <v-file-input
            label="Upload Image"
            outlined
            dense
            prepend-icon="mdi-camera"
            v-model="file"
          ></v-file-input>

          <v-btn
            v-if="!loading"
            class="mb-4"
            @click="createProject()"
            block
            color="primary"
            :disabled=isDisabled
          >
            Create
          </v-btn>

          <div class="text-center" v-else>
            <v-progress-circular
              indeterminate
              color="primary"
            ></v-progress-circular>
          </div>
        </form>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import fleekStorage from '@fleekhq/fleek-storage-js'

import { FLEEK_APIKEY, FLEEK_APISECRET } from '../config/apikeys'

export default {
  name: "AddProject",
  data: () => ({
    loading: false,
    title: "",
    description: "",
    file: null,
  }),
  computed: {
    ...mapGetters(['walletAddress', 'socialFundraiserBlockchain']),
    isDisabled() {
      return !this.title || !this.description;
    },
  },
  methods: {
    ...mapActions(['connectToBlockchain']),
    async createProject() {
      try{
        this.loading = true
        console.log(this.title, this.description, this.file)

        const uploadedFile = await fleekStorage.upload({
          apiKey: FLEEK_APIKEY,
          apiSecret: FLEEK_APISECRET,
          key: this.file.name,
          data: this.file
        });

        console.log(uploadedFile);

        await this.socialFundraiserBlockchain.methods
          .createProject(this.title, this.description, uploadedFile.hash)
          .send({ from: this.walletAddress })

        this.connectToBlockchain()
        this.loading = false
        this.$router.push('/')
      } catch(error) {
        console.log(error);
        this.loading = false
      }
    }
  }
}
</script>

<style>
  .flexRow {
    display: flex !important;
  }

  .v-input--selection-controls {
    margin-top: 5px;
    margin-left: 7px;
  }
</style>