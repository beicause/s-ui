<template>
  <v-dialog transition="dialog-bottom-transition" width="800">
    <v-card class="rounded-lg">
      <v-card-title class="d-flex align-center">
        {{ $t('actions.' + title) + " " + $t('basic.httpClient.title') }}
      </v-card-title>
      <v-divider></v-divider>
      <v-card-text style="padding: 0 16px;">
        <v-row>
          <v-col cols="12" sm="4">
            <!-- The tag is how everything else points at this client. -->
            <v-text-field
              hide-details
              :label="$t('objects.tag')"
              :error="tagError.length > 0"
              v-model="client.tag">
            </v-text-field>
          </v-col>
          <v-col cols="12" sm="4">
            <v-select
              hide-details
              :label="$t('basic.httpClient.version')"
              :items="versions"
              @update:model-value="changeVersion"
              v-model="client.version">
            </v-select>
          </v-col>
          <v-col cols="12" sm="4">
            <v-select
              hide-details
              :label="$t('basic.httpClient.engine')"
              :items="engines"
              clearable
              @click:clear="delete client.engine"
              v-model="client.engine">
            </v-select>
          </v-col>
        </v-row>
        <v-row v-if="tagError.length > 0">
          <v-col cols="12" class="text-error text-caption">{{ tagError }}</v-col>
        </v-row>
        <v-row v-if="client.version != undefined && client.version > 0">
          <v-col cols="12" sm="6" align-self="center">
            <v-switch
              v-model="client.disable_version_fallback"
              color="primary"
              :label="$t('basic.httpClient.noFallback')"
              hide-details>
            </v-switch>
          </v-col>
        </v-row>

        <QuicFields v-if="fieldGroup != 'none'" :data="client" :quic="fieldGroup == 'quic'" />
        <Headers :data="client" />
        <Dial :dial="client" mode="client" />
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="primary" variant="outlined" @click="closeModal">{{ $t('actions.close') }}</v-btn>
        <v-btn color="primary" variant="tonal" :loading="loading" :disabled="tagError.length > 0" @click="saveChanges">
          {{ $t('actions.save') }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script lang="ts">
import Dial from '@/components/Dial.vue'
import Headers from '@/components/Headers.vue'
import QuicFields from '@/components/QuicFields.vue'
import RandomUtil from '@/plugins/randomUtil'
import { i18n } from '@/locales'
import { HttpClient, HttpVersion, createHttpClient, httpFieldGroup } from '@/types/httpClient'

// The tuning fields belong to one transport or the other, so switching version
// drops the ones that no longer apply.
const versionKeys = [
  'idle_timeout',
  'keep_alive_period',
  'stream_receive_window',
  'connection_receive_window',
  'max_concurrent_streams',
  'initial_packet_size',
  'disable_path_mtu_discovery',
]

export default {
  props: ['visible', 'data', 'index', 'tags'],
  emits: ['close', 'save'],
  components: { Dial, Headers, QuicFields },
  data() {
    return {
      title: "add",
      loading: false,
      client: <HttpClient>createHttpClient(''),
      engines: [
        { title: 'Go', value: 'go' },
        { title: 'Apple', value: 'apple' },
      ],
      versions: [
        { title: 'Auto', value: 0 },
        { title: 'HTTP/1.1', value: 1 },
        { title: 'HTTP/2', value: 2 },
        { title: 'HTTP/3', value: 3 },
      ],
    }
  },
  methods: {
    updateData() {
      if (this.$props.index != -1) {
        this.title = "edit"
        this.client = <HttpClient>JSON.parse(this.$props.data)
      } else {
        this.title = "add"
        this.client = createHttpClient("hc-" + RandomUtil.randomSeq(3))
      }
    },
    changeVersion(version: HttpVersion) {
      const group = httpFieldGroup(version)
      if (group == 'none') {
        versionKeys.forEach(key => delete (<any>this.client)[key])
        return
      }
      if (group == 'http2') {
        delete this.client.initial_packet_size
        delete this.client.disable_path_mtu_discovery
      }
    },
    closeModal() {
      this.$emit('close')
    },
    saveChanges() {
      this.loading = true
      this.$emit('save', this.client)
      this.loading = false
    },
  },
  computed: {
    fieldGroup(): string {
      return httpFieldGroup(this.client.version)
    },
    tagError(): string {
      const tag = this.client.tag?.trim() ?? ''
      if (tag.length == 0) return i18n.global.t('error.invalidData') + ": " + i18n.global.t('objects.tag')
      const others = <string[]>(this.$props.tags ?? [])
      const taken = this.$props.index == -1
        ? others
        : others.filter((_, i) => i != this.$props.index)
      if (taken.includes(tag)) return i18n.global.t('error.dplData') + ": " + i18n.global.t('objects.tag')
      return ''
    },
  },
  watch: {
    visible(newValue) {
      if (newValue) this.updateData()
    },
  },
}
</script>
