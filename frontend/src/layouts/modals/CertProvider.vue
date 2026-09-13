<template>
  <v-dialog transition="dialog-bottom-transition" width="800">
    <v-card class="rounded-lg">
      <v-card-title class="d-flex align-center">
        {{ $t('actions.' + title) + " " + $t('tls.provider.title') }}
        <v-spacer></v-spacer>
        <DocLink section="tls" />
      </v-card-title>
      <v-divider></v-divider>
      <v-card-text style="padding: 0 16px;">
        <v-row>
          <v-col cols="12" sm="6">
            <v-select
              hide-details
              :label="$t('type')"
              :items="types"
              @update:model-value="changeType($event)"
              v-model="provider.type">
            </v-select>
          </v-col>
          <v-col cols="12" sm="6">
            <!-- The tag is how a TLS config points at this provider, so it has
                 to be set and unique. -->
            <v-text-field
              hide-details
              :label="$t('objects.tag')"
              :error="tagError.length > 0"
              v-model="provider.tag">
            </v-text-field>
          </v-col>
        </v-row>
        <v-row v-if="tagError.length > 0">
          <v-col cols="12" class="text-error text-caption">{{ tagError }}</v-col>
        </v-row>

        <AcmeVue v-if="provider.type == 'acme'" :data="provider" />
        <OriginCaVue v-else-if="provider.type == 'cloudflare-origin-ca'" :data="provider" />
        <v-row v-else-if="provider.type == 'tailscale'">
          <v-col cols="12">
            <v-text-field
              :label="$t('tls.provider.endpoint')"
              :hint="$t('tls.provider.endpointHint')"
              persistent-hint
              clearable
              @click:clear="delete provider.endpoint"
              v-model="provider.endpoint">
            </v-text-field>
          </v-col>
        </v-row>
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
import AcmeVue from '@/components/tls/Acme.vue'
import OriginCaVue from '@/components/tls/OriginCa.vue'
import DocLink from '@/components/DocLink.vue'
import Data from '@/store/modules/data'
import RandomUtil from '@/plugins/randomUtil'
import { i18n } from '@/locales'
import { certProvider, CertProviderType, createCertProvider } from '@/types/tls'

export default {
  props: ['visible', 'data', 'index', 'tags'],
  emits: ['close', 'save'],
  components: { AcmeVue, OriginCaVue, DocLink },
  data() {
    return {
      title: "add",
      loading: false,
      provider: <certProvider>createCertProvider('acme'),
    }
  },
  methods: {
    updateData() {
      if (this.$props.index != -1) {
        this.title = "edit"
        this.provider = <certProvider>JSON.parse(this.$props.data)
      } else {
        this.title = "add"
        this.provider = createCertProvider('acme', "cert-" + RandomUtil.randomSeq(3))
      }
    },
    // Each type has its own option set, so the fields of the one being left
    // behind are dropped rather than carried over as unknown keys.
    changeType(type: CertProviderType) {
      this.provider = createCertProvider(type, this.provider.tag)
    },
    closeModal() {
      this.$emit('close')
    },
    saveChanges() {
      this.loading = true
      this.$emit('save', this.provider)
      this.loading = false
    },
  },
  computed: {
    // ACME is hidden on Windows, where it does not work (#1189), but an
    // existing one stays editable rather than silently changing type.
    types(): { title: string; value: CertProviderType }[] {
      const all: { title: string; value: CertProviderType }[] = [
        { title: 'ACME', value: 'acme' },
        { title: 'Tailscale', value: 'tailscale' },
        { title: 'Cloudflare Origin CA', value: 'cloudflare-origin-ca' },
      ]
      if (Data().os === 'windows' && this.provider.type !== 'acme') {
        return all.filter(t => t.value !== 'acme')
      }
      return all
    },
    tagError(): string {
      const tag = this.provider.tag?.trim() ?? ''
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
