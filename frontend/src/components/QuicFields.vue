<template>
  <v-card :subtitle="quic ? 'QUIC' : 'HTTP/2'" style="background-color: inherit;">
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-switch v-model="show" color="primary" :label="$t('enable')" hide-details></v-switch>
      </v-col>
    </v-row>
    <template v-if="show">
      <v-row>
        <v-col cols="12" sm="6" md="4">
          <v-text-field
            :label="$t('quic.idleTimeout')"
            placeholder="30s"
            hide-details
            clearable
            v-model="idleTimeout">
          </v-text-field>
        </v-col>
        <v-col cols="12" sm="6" md="4">
          <v-text-field
            :label="$t('quic.keepAlive')"
            placeholder="0s"
            hide-details
            clearable
            v-model="keepAlivePeriod">
          </v-text-field>
        </v-col>
        <v-col cols="12" sm="6" md="4">
          <v-text-field
            :label="$t('quic.maxStreams')"
            type="number"
            min="0"
            hide-details
            v-model.number="maxConcurrentStreams">
          </v-text-field>
        </v-col>
      </v-row>
      <!-- Byte sizes take a plain number or a string with a unit, e.g. 8mb. -->
      <v-row>
        <v-col cols="12" sm="6">
          <v-text-field
            :label="$t('quic.streamWindow')"
            placeholder="8mb"
            hide-details
            clearable
            v-model="streamReceiveWindow">
          </v-text-field>
        </v-col>
        <v-col cols="12" sm="6">
          <v-text-field
            :label="$t('quic.connectionWindow')"
            placeholder="16mb"
            hide-details
            clearable
            v-model="connectionReceiveWindow">
          </v-text-field>
        </v-col>
      </v-row>
      <v-row v-if="quic">
        <v-col cols="12" sm="6" md="4">
          <v-text-field
            :label="$t('quic.initialPacketSize')"
            type="number"
            min="0"
            hide-details
            v-model.number="initialPacketSize">
          </v-text-field>
        </v-col>
        <v-col cols="12" sm="6" md="4" align-self="center">
          <v-switch
            v-model="data.disable_path_mtu_discovery"
            color="primary"
            :label="$t('quic.disableMtuDiscovery')"
            hide-details>
          </v-switch>
        </v-col>
      </v-row>
    </template>
  </v-card>
</template>

<script lang="ts">
// The transport tuning sing-box flattens into whatever carries it. Every field
// is optional, and an empty one is removed rather than written out: sing-box
// rejects "" for a duration or a byte size, so a revealed but unfilled group
// has to leave no trace.
const http2Keys = [
  'idle_timeout',
  'keep_alive_period',
  'stream_receive_window',
  'connection_receive_window',
  'max_concurrent_streams',
]
const quicKeys = ['initial_packet_size', 'disable_path_mtu_discovery']

export default {
  props: ['data', 'quic'],
  data() {
    return {
      // Revealing the group writes nothing, so clearing the last field does
      // not make it collapse under the operator.
      show: false,
    }
  },
  methods: {
    syncShow() {
      this.show = this.fields.some(key => this.$props.data?.[key] != undefined)
    },
    text(key: string): string {
      return this.$props.data?.[key] ?? ''
    },
    setText(key: string, value: string) {
      const trimmed = (value ?? '').trim()
      if (trimmed) this.$props.data[key] = trimmed
      else delete this.$props.data[key]
    },
  },
  computed: {
    fields(): string[] {
      return this.$props.quic ? http2Keys.concat(quicKeys) : http2Keys
    },
    idleTimeout: {
      get(): string { return this.text('idle_timeout') },
      set(v: string) { this.setText('idle_timeout', v) }
    },
    keepAlivePeriod: {
      get(): string { return this.text('keep_alive_period') },
      set(v: string) { this.setText('keep_alive_period', v) }
    },
    streamReceiveWindow: {
      get(): string { return String(this.$props.data?.stream_receive_window ?? '') },
      set(v: string) { this.setText('stream_receive_window', v) }
    },
    connectionReceiveWindow: {
      get(): string { return String(this.$props.data?.connection_receive_window ?? '') },
      set(v: string) { this.setText('connection_receive_window', v) }
    },
    maxConcurrentStreams: {
      get(): number { return this.$props.data?.max_concurrent_streams ?? 0 },
      set(v: number) {
        if (v > 0) this.$props.data.max_concurrent_streams = v
        else delete this.$props.data.max_concurrent_streams
      }
    },
    initialPacketSize: {
      get(): number { return this.$props.data?.initial_packet_size ?? 0 },
      set(v: number) {
        if (v > 0) this.$props.data.initial_packet_size = v
        else delete this.$props.data.initial_packet_size
      }
    },
  },
  watch: {
    // Turning the group off clears what it owns; the fields themselves are
    // already removed as they are emptied.
    show(v: boolean) {
      if (!v) this.fields.forEach(key => delete this.$props.data[key])
    },
    data: {
      handler() { this.syncShow() },
      immediate: true,
    },
  },
}
</script>
