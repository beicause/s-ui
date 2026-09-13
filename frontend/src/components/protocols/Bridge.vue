<template>
  <v-card>
    <v-card-subtitle v-if="direction != 'out_json'">Bridge</v-card-subtitle>
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.bridge.interface')"
          hide-details
          clearable
          @click:clear="delete data.interface"
          v-model="data.interface">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.bridge.bridgeName')"
          hide-details
          clearable
          @click:clear="delete data.bridge_name"
          v-model="data.bridge_name">
        </v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.bridge.tableIndex')"
          type="number"
          min="0"
          hide-details
          v-model.number="tableIndex">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.bridge.ruleIndex')"
          type="number"
          min="0"
          hide-details
          v-model.number="ruleIndex">
        </v-text-field>
      </v-col>
    </v-row>
  </v-card>
</template>

<script lang="ts">
export default {
  props: ['data', 'direction'],
  computed: {
    tableIndex: {
      get(): number { return this.$props.data?.iproute2_table_index ?? 0 },
      set(v: number) { this.$props.data.iproute2_table_index = v > 0 ? v : undefined }
    },
    ruleIndex: {
      get(): number { return this.$props.data?.iproute2_rule_index ?? 0 },
      set(v: number) { this.$props.data.iproute2_rule_index = v > 0 ? v : undefined }
    },
  },
}
</script>
