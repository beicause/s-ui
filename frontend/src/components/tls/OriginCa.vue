<template>
  <v-card subtitle="Cloudflare Origin CA" style="background-color: inherit;">
    <v-row>
      <v-col cols="12">
        <v-text-field
          :label="$t('rule.domain') + ' ' + $t('commaSeparated')"
          hide-details
          v-model="domains">
        </v-text-field>
      </v-col>
    </v-row>
    <!-- Either credential works; the API token is the narrower of the two. -->
    <v-row>
      <v-col cols="12" sm="6">
        <v-text-field
          :label="$t('tls.provider.apiToken')"
          type="password"
          hide-details
          clearable
          @click:clear="delete data.api_token"
          v-model="data.api_token">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6">
        <v-text-field
          :label="$t('tls.provider.originCaKey')"
          type="password"
          hide-details
          clearable
          @click:clear="delete data.origin_ca_key"
          v-model="data.origin_ca_key">
        </v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-select
          :label="$t('tls.provider.requestType')"
          :items="requestTypes"
          hide-details
          clearable
          @click:clear="delete data.request_type"
          v-model="data.request_type">
        </v-select>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-select
          :label="$t('tls.provider.validity')"
          :items="validities"
          hide-details
          clearable
          @click:clear="delete data.requested_validity"
          v-model="data.requested_validity">
        </v-select>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('tls.acme.dataDir')"
          hide-details
          clearable
          @click:clear="delete data.data_directory"
          v-model="data.data_directory">
        </v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-select
          :label="$t('basic.httpClient.title')"
          :items="httpClients"
          :no-data-text="$t('basic.httpClient.none')"
          hide-details
          clearable
          @click:clear="delete data.http_client"
          v-model="data.http_client">
        </v-select>
      </v-col>
    </v-row>
  </v-card>
</template>

<script lang="ts">
import { httpClientTags } from '@/plugins/httpClient'

export default {
  props: ['data'],
  data() {
    return {
      requestTypes: [
        { title: 'RSA', value: 'origin-rsa' },
        { title: 'ECC', value: 'origin-ecc' },
      ],
      // The only validities Cloudflare accepts, in days.
      validities: [7, 30, 90, 365, 730, 1095, 5475],
    }
  },
  computed: {
    httpClients(): string[] {
      return httpClientTags()
    },
    domains: {
      get(): string { return this.$props.data.domain ? this.$props.data.domain.join(',') : '' },
      set(v: string) {
        if (!v.endsWith(',')) {
          this.$props.data.domain = v.length > 0 ? v.split(',') : []
        }
      }
    },
  },
}
</script>
