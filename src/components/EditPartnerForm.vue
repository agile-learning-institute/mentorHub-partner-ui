<template>
  <v-container>
    <h1>Edit Partner</h1>
    <v-form>
      <v-text-field label="Partner Name" @change="saveMe($event, 'partnerName')" v-model="partner.name" required></v-text-field>
      <!-- TODO: Find out what to do with URL since api does not return it -->
      <!-- <v-select label="URL" @update:model-value="saveMe($event, 'url')" v-model="partner.url" :items="partners.url"></v-select> -->
      <v-text-field label="Notes" @change="saveMe($event, 'description')" :rules="[rules.descriptionCount]" v-model="partner.description"></v-text-field>

      <v-select label="Status" @update:model-value="saveMe($event, 'status')" v-model="partner.status" :items="[...partners.status]"></v-select>

      <EditContactsForm :save-me="saveMe" :rules="rules" />
    </v-form>
  </v-container>
</template>

<script setup lang="ts">
import { computed, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import { usePartnersStore } from "@/stores/index";
import { type Partner } from "@/stores/index";
import { storeToRefs } from "pinia";
import EditContactsForm from "./EditContactsForm.vue";

const route = useRoute();
const router = useRouter();

const store = usePartnersStore();
const { partner, config } = storeToRefs(store);
const { getPartner, patchPartner, selectPerson } = store;

const partners = computed(() => config.value?.enumerators ? {
  status: Object.keys(config.value?.enumerators.partnerStatus as Partner),
} : {});

const rules = {
  required: (value: string) => !!value || "This is a required field",
  nameCount: (value: string) => value?.length <= 32 || "Max 32 characters for name field",
  descriptionCount: (value: string) => value?.length <= 256 || "Max 256 characters for description field",
}

onMounted(() => {
  getPartner(route.params.id as string);
  // Check for the person id parameter
  if (route.query.person_id) {
    const thePerson = { _id: route.query.person_id };
    const thePartner = { _id: route.params.id };
    selectPerson(thePerson, thePartner).then(() => {
      router.replace({
        path: route.path,
        query: {}
      })
    });
  }
})

async function saveMe(event: any, fieldName: string) {
  const value = typeof event === "string" || Array.isArray(event) ? event
    : event.name && event._id ? event._id
    : event.target.value;

  try {
    await patchPartner({
      id: route.params.id,
      fieldName,
      value
    });
  } catch (error: any) {
    alert(error?.message);
    event.target?.focus();
  }
}
</script>

<style scoped>

.flex-container {
display: flex;
justify-content: space-between; /* Distributes the items evenly across the container */
align-items: center; /* Aligns the items vertically in the center */
}

.flex-item {
flex: 1; /* Each item takes an equal share of the container */
}

</style>