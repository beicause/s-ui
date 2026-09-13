<template>
  <v-card subtitle="OOM Killer">
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.oomKiller.memoryLimit')"
          hide-details
          clearable
          placeholder="1gb"
          @click:clear="delete data.memory_limit"
          v-model="data.memory_limit">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.oomKiller.safetyMargin')"
          hide-details
          clearable
          placeholder="128mb"
          @click:clear="delete data.safety_margin"
          v-model="data.safety_margin">
        </v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.oomKiller.minInterval')"
          hide-details
          clearable
          placeholder="10s"
          @click:clear="delete data.min_interval"
          v-model="data.min_interval">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.oomKiller.maxInterval')"
          hide-details
          clearable
          placeholder="1m"
          @click:clear="delete data.max_interval"
          v-model="data.max_interval">
        </v-text-field>
      </v-col>
    </v-row>
  </v-card>
</template>

<script lang="ts">
export default {
  props: ['data'],
}
</script>
