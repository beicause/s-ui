<template>
  <v-card>
    <v-card-subtitle>Cloudflared</v-card-subtitle>
    <v-row>
      <v-col cols="12" md="8">
        <v-text-field
          :label="$t('types.cloudflared.token')"
          hide-details
          type="password"
          v-model="data.token">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-select
          hide-details
          :label="$t('types.cloudflared.protocol')"
          :items="protocols"
          clearable
          @click:clear="delete data.protocol"
          v-model="data.protocol">
        </v-select>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.cloudflared.haConnections')"
          type="number"
          min="0"
          hide-details
          v-model.number="haConnections">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-select
          hide-details
          :label="$t('types.cloudflared.edgeIpVersion')"
          :items="edgeIpVersions"
          clearable
          @click:clear="delete data.edge_ip_version"
          v-model="data.edge_ip_version">
        </v-select>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-select
          hide-details
          :label="$t('types.cloudflared.datagramVersion')"
          :items="datagramVersions"
          clearable
          @click:clear="delete data.datagram_version"
          v-model="data.datagram_version">
        </v-select>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.cloudflared.region')"
          hide-details
          clearable
          @click:clear="delete data.region"
          v-model="data.region">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.cloudflared.gracePeriod')"
          hide-details
          clearable
          placeholder="30s"
          @click:clear="delete data.grace_period"
          v-model="data.grace_period">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-switch v-model="data.post_quantum" color="primary" :label="$t('types.cloudflared.postQuantum')" hide-details></v-switch>
      </v-col>
    </v-row>
  </v-card>
</template>

<script lang="ts">
export default {
  props: ['data'],
  data() {
    return {
      protocols: [
        { title: 'Auto', value: 'auto' },
        { title: 'QUIC', value: 'quic' },
        { title: 'HTTP/2', value: 'http2' },
        { title: 'h2mux', value: 'h2mux' },
      ],
      edgeIpVersions: [
        { title: 'Auto', value: 0 },
        { title: 'IPv4', value: 4 },
        { title: 'IPv6', value: 6 },
      ],
      datagramVersions: [
        { title: 'v2', value: 'v2' },
        { title: 'v3', value: 'v3' },
      ],
    }
  },
  computed: {
    haConnections: {
      get(): number { return this.$props.data?.ha_connections ?? 0 },
      set(v: number) { this.$props.data.ha_connections = v > 0 ? v : undefined }
    },
  },
}
</script>
