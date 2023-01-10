<template>
  <a-modal
    v-model="isVisible"
    :footer="null"
    :title="sectionName"
    :afterClose="closeModal"
  >
    <vuci-form :uci-config="configFile" @applied="$emit('close')">
      <vuci-named-section :name="sectionId" v-slot="{ s }">
        <vuci-form-item-select
          key="datachanged"
          :uci-section="s"
          label="Protocol"
          name="proto"
          :options="proto"
          initial="static"
          @change="onChange($event)"
          required
        />
        <vuci-form-item-input
          :uci-section="s"
          label="IP address"
          name="address"
          depend="proto == 'static'"
          rules="ipaddr"
        />
        <vuci-form-item-input
          :uci-section="s"
          label="Netmask"
          name="netmask"
          depend="proto == 'static'"
          rules="netmask4"
        />
        <vuci-form-item-input
          :uci-section="s"
          label="Gateway"
          name="gateway"
          depend="proto == 'static'"
          rules="ipaddr"
        />
        <vuci-form-item-list
          :uci-section="s"
          label="DNS"
          name="dns"
          depend="proto == 'static'"
          rules="ipaddr"
        />
      </vuci-named-section>
    </vuci-form>
  </a-modal>
</template>
<script>
export default {
  name: 'UciModal',
  watch: {
    isVisible: {
      immediate: true,
      handler: function (newVal) {
        this.isVisible = newVal
      }
    }
  },

  data () {
    return {
      proto: ['dhcp', 'static'],
      isVisible: this.visible
    }
  },

  props: {
    visible: Boolean,
    configFile: String,
    sectionId: String,
    sectionName: String,
  },

  methods: {
    onChange (event) {
      if (event.model === 'dhcp') {
        this.$uci.set(this.configFile, this.sectionId, 'address', '')
        this.$uci.set(this.configFile, this.sectionId, 'netmask', '')
        this.$uci.set(this.configFile, this.sectionId, 'gateway', '')
        this.$uci.set(this.configFile, this.sectionId, 'dns', '')
      } else {
        this.$uci.reset()
      }
    },
    closeModal () {
      this.$emit('close')
    }
  }
}
</script>
