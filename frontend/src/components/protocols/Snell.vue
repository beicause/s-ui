<template>
  <v-card>
    <v-card-subtitle v-if="direction != 'out_json'">Snell</v-card-subtitle>
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-select
          hide-details
          :label="$t('types.snell.version')"
          :items="versions"
          v-model="version">
        </v-select>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.snell.psk')"
          hide-details
          v-model="data.psk"
          :append-inner-icon="direction == 'in' ? 'mdi-refresh' : undefined"
          @click:append-inner="generatePsk">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4" v-if="direction === 'out'">
        <v-text-field
          :label="$t('types.snell.userKey')"
          hide-details
          clearable
          @click:clear="delete data.userkey"
          v-model="data.userkey">
        </v-text-field>
      </v-col>
    </v-row>
    <!-- v5 (inbound) and v4 (outbound) carry obfs; v6 carries a mode -->
    <v-row v-if="obfsVersion">
      <v-col cols="12" sm="6" md="4">
        <v-select
          hide-details
          :label="$t('types.snell.obfsMode')"
          :items="obfsModes"
          clearable
          @click:clear="delete data.obfs_mode"
          v-model="data.obfs_mode">
        </v-select>
      </v-col>
      <v-col cols="12" sm="6" md="4" v-if="direction === 'out'">
        <v-text-field
          :label="$t('types.snell.obfsHost')"
          hide-details
          clearable
          @click:clear="delete data.obfs_host"
          v-model="data.obfs_host">
        </v-text-field>
      </v-col>
    </v-row>
    <v-row v-if="data.version == 6">
      <v-col cols="12" sm="6" md="4">
        <v-select
          hide-details
          :label="$t('types.snell.mode')"
          :items="modes"
          clearable
          @click:clear="delete data.mode"
          v-model="data.mode">
        </v-select>
      </v-col>
    </v-row>
    <v-row v-if="direction === 'out'">
      <v-col cols="12" sm="6" md="4">
        <v-switch v-model="data.reuse" color="primary" :label="$t('types.snell.reuse')" hide-details></v-switch>
      </v-col>
    </v-row>
  </v-card>
</template>

<script lang="ts">
import RandomUtil from '@/plugins/randomUtil'

export default {
  props: ['data', 'direction'],
  data() {
    return {
      obfsModes: [
        { title: 'None', value: 'none' },
        { title: 'HTTP', value: 'http' },
        { title: 'TLS', value: 'tls' },
      ],
      modes: [
        { title: 'Default', value: 'default' },
        { title: 'Unshaped', value: 'unshaped' },
        { title: 'Unsafe Raw', value: 'unsafe-raw' },
      ],
    }
  },
  methods: {
    generatePsk() {
      // sing-box requires 12-255 bytes.
      this.$props.data.psk = RandomUtil.randomSeq(32)
    },
  },
  computed: {
    // Inbounds support v5 and v6, outbounds v4 and v6.
    versions(): { title: string; value: number }[] {
      const legacy = this.$props.direction === 'in' ? 5 : 4
      return [
        { title: 'v' + legacy, value: legacy },
        { title: 'v6', value: 6 },
      ]
    },
    obfsVersion(): boolean {
      const legacy = this.$props.direction === 'in' ? 5 : 4
      return this.$props.data.version === legacy
    },
    version: {
      get(): number { return this.$props.data.version },
      // The version selects which extra options apply, so drop the ones
      // belonging to the version being left behind.
      set(v: number) {
        this.$props.data.version = v
        if (v === 6) {
          delete this.$props.data.obfs_mode
          delete this.$props.data.obfs_host
        } else {
          delete this.$props.data.mode
        }
      }
    },
  },
}
</script>
