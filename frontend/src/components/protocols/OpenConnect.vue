<template>
  <v-card subtitle="OpenConnect">
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.openconnect.server')"
          hide-details
          placeholder="vpn.example.com"
          v-model="data.server">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-select
          hide-details
          :label="$t('types.openconnect.flavor')"
          :items="flavors"
          v-model="data.flavor">
        </v-select>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.openconnect.interfaceName')"
          hide-details
          clearable
          @click:clear="delete data.name"
          v-model="data.name">
        </v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.un')"
          hide-details
          clearable
          @click:clear="delete data.username"
          v-model="data.username">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.pw')"
          hide-details
          type="password"
          clearable
          @click:clear="delete data.password"
          v-model="data.password">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.openconnect.authGroup')"
          hide-details
          clearable
          @click:clear="delete data.auth_group"
          v-model="data.auth_group">
        </v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <v-text-field
          :label="$t('types.openconnect.cookie')"
          hide-details
          type="password"
          clearable
          @click:clear="delete data.cookie"
          v-model="data.cookie">
        </v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          label="MTU"
          type="number"
          min="0"
          hide-details
          v-model.number="mtu">
        </v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-text-field
          :label="$t('types.openconnect.udpTimeout')"
          hide-details
          clearable
          placeholder="5m"
          @click:clear="delete data.udp_timeout"
          v-model="data.udp_timeout">
        </v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" sm="6" md="4">
        <v-switch v-model="data.system" color="primary" :label="$t('types.openconnect.system')" hide-details></v-switch>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-switch v-model="data.no_udp" color="primary" :label="$t('types.openconnect.noUdp')" hide-details></v-switch>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-switch v-model="data.ipv6_disabled" color="primary" :label="$t('types.openconnect.ipv6Disabled')" hide-details></v-switch>
      </v-col>
    </v-row>

  </v-card>
</template>

<script lang="ts">
export default {
  props: ['data'],
  data() {
    return {
      flavors: [
        { title: 'Cisco AnyConnect', value: 'anyconnect' },
        { title: 'Palo Alto GlobalProtect', value: 'gp' },
        { title: 'Fortinet', value: 'fortinet' },
        { title: 'F5 BIG-IP', value: 'f5' },
        { title: 'Juniper Pulse', value: 'pulse' },
        { title: 'Junos Network Connect', value: 'nc' },
      ],
    }
  },
  computed: {
    mtu: {
      get(): number { return this.$props.data?.mtu ?? 0 },
      set(v: number) { this.$props.data.mtu = v > 0 ? v : undefined }
    },
  },
}
</script>
