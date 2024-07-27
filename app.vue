<script setup>
const search = ref({
  'location': null,
  'specialties': []
})
const chosenSpeciality = ref(null)
const availabledBedsOnly = ref(false);

// Requête de tous les hopitaux
const {data: hospitals} = await useFetch('http://localhost:9090/hospital')

// Requête de toutes les spécialitées pour le menu
const {data: specialities} = await useFetch('http://localhost:9090/hospital/specialities')

const searchForHospital = () => {
  console.log(chosenSpeciality.value)
  let baseUrl = '/hospital'
  if (availabledBedsOnly) {
    baseUrl = '/hospital/available'
  }

  if (chosenSpeciality.value) {
    baseUrl = baseUrl + '/speciality/' + chosenSpeciality.value
  }

  console.log(baseUrl)
}

const formatSpecialities = (specialities) => {
  return specialities.split(',')
      .map(spec => spec.trim())
      .map(spec => spec.charAt(0).toUpperCase() + spec.slice(1))
      .join(' / ');
}

const getBedAvailabilityClass = (availableBeds) => {
  if (availableBeds < 5) {
    return 'text-red-500';
  } else if (availableBeds >= 5 && availableBeds <= 25) {
    return 'text-orange-500';
  } else {
    return 'text-green-500';
  }
}
</script>

<template>
  <div class="flex flex-col items-center justify-center min-h-screen bg-gradient-to-br from-cyan-300 to-violet-400">
    <h2 class="font-semibold text-4xl mb-10">MEDHEAD URGENCY</h2>
    <div class="w-full max-w-md p-4 rounded-lg bg-blue-200 shadow-md">
      <UForm :state="search" @submit="searchForHospital()">
        <UFormGroup>
          <template #label>
            <h2 class="text-white text-xl font-semibold mt-5 w-full">Current Location</h2>
          </template>
          <UInput icon="i-heroicons-map-pin" class="text-white w-full" size="xl"/>
        </UFormGroup>
        <UFormGroup>
          <template #label>
            <h2 class="text-white text-xl font-semibold mt-5">What kind of treatment is needed ?</h2>
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
          <UToggle v-model="availabledBedsOnly" size="xl" />
        </UFormGroup>
        <div class="flex justify-center mt-4">
          <UButton color="black" class="text-xl font-semibold uppercase" type="submit">Search</UButton>
        </div>
      </UForm>
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
