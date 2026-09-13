<template>
  <v-card subtitle="API">
    <v-row>
      <v-col cols="12" sm="6">
        <v-text-field
          :label="$t('types.api.secret')"
          hide-details
          type="password"
          clearable
          @click:clear="delete data.secret"
          v-model="data.secret">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" align-self="center">
        <v-switch
          v-model="data.access_control_allow_private_network"
          color="primary"
          :label="$t('types.api.allowPrivateNetwork')"
          hide-details>
        </v-switch>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <v-combobox
          :label="$t('types.api.allowOrigin')"
          hide-details
          multiple
          chips
          closable-chips
          clearable
          @click:clear="delete data.access_control_allow_origin"
          v-model="data.access_control_allow_origin">
        </v-combobox>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" sm="6" align-self="center">
        <v-switch
          v-model="dashboardEnabled"
          color="primary"
          :label="$t('types.api.dashboard')"
          hide-details>
        </v-switch>
      </v-col>
      <v-col cols="12" sm="6" v-if="dashboardEnabled">
        <v-text-field
          :label="$t('types.api.dashboardPath')"
          hide-details
          clearable
          v-model="dashboardPath">
        </v-text-field>
      </v-col>
    </v-row>
    <!-- Letting sing-box fetch the dashboard itself needs the full object
         form, so it stays behind a toggle: without it the setting keeps the
         short boolean or path shape. -->
    <template v-if="dashboardEnabled">
      <v-row>
        <v-col cols="12" sm="6" align-self="center">
          <v-switch
            v-model="dashboardDownload"
            color="primary"
            :label="$t('types.api.dashboardDownload')"
            hide-details>
          </v-switch>
        </v-col>
      </v-row>
      <v-row v-if="dashboardDownload">
        <v-col cols="12">
          <v-text-field
            :label="$t('types.api.dashboardUrl')"
            hide-details
            v-model="dashboardDownloadUrl">
          </v-text-field>
        </v-col>
        <v-col cols="12" sm="6">
          <v-select
            :label="$t('basic.httpClient.title')"
            :items="httpClients"
            :no-data-text="$t('basic.httpClient.none')"
            hide-details
            clearable
            v-model="dashboardHttpClient">
          </v-select>
        </v-col>
        <v-col cols="12" sm="6">
          <v-text-field
            :label="$t('ruleset.interval')"
            :suffix="$t('date.d')"
            type="number"
            min="0"
            hide-details
            v-model.number="dashboardInterval">
          </v-text-field>
        </v-col>
      </v-row>
    </template>
  </v-card>
</template>

<script lang="ts">
import { httpClientTags, refTag } from '@/plugins/httpClient'

export default {
  props: ['data'],
  methods: {
    // dashboard accepts a bool, a path string or an object. Anything beyond
    // enabled and path needs the object, so the short forms are widened on
    // demand rather than up front.
    asObject(): any {
      const dashboard = this.$props.data?.dashboard
      if (dashboard && typeof dashboard === 'object') return dashboard
      const widened: any = { enabled: true }
      if (typeof dashboard === 'string' && dashboard.length > 0) widened.path = dashboard
      this.$props.data.dashboard = widened
      return widened
    },
  },
  computed: {
    httpClients(): string[] {
      return httpClientTags()
    },
    dashboardEnabled: {
      get(): boolean {
        const dashboard = this.$props.data?.dashboard
        if (typeof dashboard === 'boolean') return dashboard
        if (typeof dashboard === 'string') return dashboard.length > 0
        return dashboard?.enabled === true
      },
      set(v: boolean) {
        if (!v) {
          delete this.$props.data.dashboard
          return
        }
        this.$props.data.dashboard = true
      }
    },
    dashboardPath: {
      get(): string {
        const dashboard = this.$props.data?.dashboard
        if (typeof dashboard === 'string') return dashboard
        if (dashboard && typeof dashboard === 'object') return dashboard.path ?? ''
        return ''
      },
      set(v: string) {
        const dashboard = this.$props.data?.dashboard
        if (dashboard && typeof dashboard === 'object') {
          if (v) dashboard.path = v
          else delete dashboard.path
          return
        }
        this.$props.data.dashboard = v ? v : true
      }
    },
    // Turning this off drops the download settings and lets the value fall
    // back to its short form.
    dashboardDownload: {
      get(): boolean {
        const dashboard = this.$props.data?.dashboard
        if (!dashboard || typeof dashboard !== 'object') return false
        return dashboard.download_url != undefined
          || dashboard.http_client != undefined
          || dashboard.update_interval != undefined
      },
      set(v: boolean) {
        if (v) {
          this.asObject().download_url = ''
          return
        }
        const dashboard = this.$props.data?.dashboard
        if (!dashboard || typeof dashboard !== 'object') return
        const path = dashboard.path
        this.$props.data.dashboard = path ? path : true
      }
    },
    dashboardDownloadUrl: {
      get(): string { return this.$props.data?.dashboard?.download_url ?? '' },
      set(v: string) {
        const dashboard = this.asObject()
        if (v) dashboard.download_url = v
        else delete dashboard.download_url
      }
    },
    dashboardHttpClient: {
      get(): string | undefined { return refTag(this.$props.data?.dashboard?.http_client) },
      set(v: string | undefined) {
        const dashboard = this.asObject()
        if (v) dashboard.http_client = v
        else delete dashboard.http_client
      }
    },
    dashboardInterval: {
      get(): number {
        const interval = this.$props.data?.dashboard?.update_interval
        return interval ? parseInt(String(interval).replace('d', '')) : 0
      },
      set(v: number) {
        const dashboard = this.asObject()
        if (v > 0) dashboard.update_interval = v + 'd'
        else delete dashboard.update_interval
      }
    },
  },
}
</script>
