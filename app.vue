<script setup>
import { ref } from 'vue';
import { useFetch } from '#imports';

const search = ref({
  'location': null,
  'specialties': []
});
const chosenSpeciality = ref(null);
const availabledBedsOnly = ref(false);
const userAddress = ref('');
const distance = ref(50);

// Requête de toutes les spécialités pour le menu
const { data: specialities } = await useFetch('http://localhost:9090/hospital/specialities');

let hospitals = ref(null);

const searchForHospital = async () => {
  const baseUrl = 'http://localhost:9090/hospital/search';
  let queryParams = [];

  if (availabledBedsOnly.value) {
    queryParams.push(`availableBeds=true`);
  }

  if (chosenSpeciality.value) {
    queryParams.push(`speciality=${encodeURIComponent(chosenSpeciality.value)}`);
  }

  if (userAddress.value) {
    const coords = await getCoordinates(userAddress.value);
    queryParams.push(`latInit=${coords.lat}`);
    queryParams.push(`longInit=${coords.lng}`);
    queryParams.push(`distance=${distance.value}`);
  }

  const queryString = queryParams.length ? '?' + queryParams.join('&') : '';
  const url = baseUrl + queryString;

  const { data } = await useFetch(url);
  hospitals.value = data.value;
};

const getCoordinates = async (address) => {
  const apiKey = 'ec307b828a5c4d7fbaea2342992a5196';
  const url = `https://api.opencagedata.com/geocode/v1/json?q=${encodeURIComponent(address)}&key=${apiKey}`;

  const { data } = await useFetch(url);

  if (data.value && data.value.results && data.value.results.length > 0) {
    const result = data.value.results[0];
    return {
      lat: result.geometry.lat,
      lng: result.geometry.lng
    };
  } else {
    throw new Error('Geocode was not successful');
  }
};

const formatSpecialities = (specialities) => {
  return specialities.split(',')
      .map(spec => spec.trim())
      .map(spec => spec.charAt(0).toUpperCase() + spec.slice(1))
      .join(' / ');
};

const getBedAvailabilityClass = (availableBeds) => {
  if (availableBeds < 5) {
    return 'text-red-500';
  } else if (availableBeds >= 5 && availableBeds <= 25) {
    return 'text-orange-500';
  } else {
    return 'text-green-500';
  }
};
</script>

<template>
  <div class="flex flex-col items-center justify-center min-h-screen bg-gradient-to-br from-cyan-300 to-violet-400">
    <h2 class="font-semibold text-4xl my-10">MEDHEAD URGENCY</h2>
    <div class="w-full max-w-md p-4 rounded-lg bg-blue-200 shadow-md">
      <UForm :state="search" @submit="searchForHospital">
        <UFormGroup>
          <template #label>
            <h2 class="text-white text-xl font-semibold mt-5 w-full">Current Location</h2>
          </template>
          <UInput v-model="userAddress" icon="i-heroicons-map-pin" class="text-white w-full" size="xl"/>
        </UFormGroup>
        <UFormGroup :disabled="!userAddress">
          <template #label>
            <h2 class="text-white text-xl font-semibold mt-5 mb-2 w-full">Search range (Current location required)</h2>
          </template>
          <URange v-model="distance" size="xl" color="blue" :disabled="!userAddress"/>
          {{distance}} KM radius
        </UFormGroup>
        <UFormGroup>
          <template #label>
            <h2 class="text-white text-xl font-semibold mt-5">What kind of treatment is needed?</h2>
          </template>
          <div class="flex items-center space-x-2">
            <USelectMenu
                searchable
                searchable-placeholder="Search ..."
                color="white"
                placeholder="Select what kind of help you need"
                :options="specialities"
                v-model="chosenSpeciality"
                size="xl"
            />
            <UButton @click="chosenSpeciality = null" color="white" class="text-xl">
              <UIcon name="i-heroicons-x-mark"/>
            </UButton>
          </div>
        </UFormGroup>
        <UFormGroup>
          <template #label>
            <h2 class="text-white text-xl font-semibold mt-5 mb-2 w-full">Search only hospital with beds available</h2>
          </template>
          <UToggle v-model="availabledBedsOnly" size="xl"/>
        </UFormGroup>
        <div class="flex justify-center mt-4">
          <UButton color="black" class="text-xl font-semibold uppercase" type="submit">Search</UButton>
        </div>
      </UForm>
    </div>
    <div>
      <h2 class="font-semibold text-2xl"><span>{{ hospitals?.length ?? 0 }}</span> results found</h2>
    </div>
    <div class="grid grid-cols-3 gap-4 mt-10 mx-4">
      <UCard v-for="hospital in hospitals" :key="hospital.id" class="bg-white dark:bg-white dark:text-black">
        <h2 class="font-bold text-xl my-1">{{ hospital.name }}</h2>
        <UDivider/>
        <p class="font-semibold my-1">
          <UIcon name="i-heroicons-map-pin"/>
          {{ hospital.address }}
        </p>
        <UDivider/>
        <p>
          <UIcon name="i-heroicons-magnifying-glass my-1"/>
          {{ formatSpecialities(hospital.speciality) }}
        </p>
        <UDivider/>
        <p>Number of beds available:
          <span class="font-semibold" :class="getBedAvailabilityClass(hospital.available_beds)">
            {{ hospital.available_beds }}
          </span>
        </p>
      </UCard>
    </div>
  </div>
</template>
