<template>
  <div class="example">
    <vuci-form uci-config="vuci_components_task" loaded>
      <vuci-typed-section type="interface" :columns="columns" :key="dataTable">
        <template #hostname="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="hostname" />
        </template>
        <template #address="{ s }">
          <vuci-form-item-dummy
            :uci-section="s"
            name="address"
            rules="ipaddr"
          />
        </template>
        <template #netmask="{ s }">
          <vuci-form-item-dummy
            :uci-section="s"
            name="netmask"
            rules="netmask4"
          />
        </template>
        <template #actions="{ s }">
          <a-space>
            <a-button type="primary" @click="editSection(s)">Edit</a-button>
            <a-popconfirm
              title="Are you sure？"
              @confirm="deleteSection(s['.name'])"
              ok-text="Yes"
              cancel-text="No"
            >
              <a-button type="danger">{{ $t("Delete") }}</a-button>
            </a-popconfirm>
          </a-space>
        </template>
      </vuci-typed-section>
      <template #footer>
        <div></div>
      </template>
    </vuci-form>
    <a-space>
      Interface name
      <a-input v-model="newInterfaceName"/>
      <a-button type="primary" @click="handleAddInterface">Add</a-button>
    </a-space>
    <uci-modal
      v-if="visible"
      :configFile="configFile"
      :sectionId="sectionId"
      :sectionName="sectionName"
      :visible="visible"
      @close="handleEditModal"
    />
  </div>
</template>

<script>
import UciModal from './components/UciModal.vue'

export default {
  components: { UciModal},
  props: {
    loaded: Boolean
  },
  data () {
    return {
      columns: [
        { name: 'hostname', label: 'Interface name' },
        { name: 'address', label: 'IP address' },
        { name: 'netmask', label: 'Netmask' },
        { name: 'actions', label: 'Actions' }
      ],
      newInterfaceName: '',
      type: 'interface',
      configFile: 'vuci_components_task',
      visible: false,
      section: '',
      sectionId: '',
      sectionName: '',
      dataTable: 0
    }
  },
  methods: {
    async load () {
      await this.$uci.load(this.configFile)
      this.reloadData()
    },

    reloadData () {
      this.dataTable++
    },

    handleEditModal () {
      this.load()
      this.visible = !this.visible
    },

    editSection (s) {
      this.handleEditModal()
      this.load()
      this.section = s
      this.sectionId = s['.name']
      this.sectionName = s.hostname
    },

    deleteSection (sectionId) {
      this.$uci.del(this.configFile, sectionId, null)
      this.$uci.save().then(() => {
        this.$uci.apply()
      })
      this.reloadData()
    },

    handleAddInterface () {
      const section = this.$uci.add(this.configFile, this.type)
      this.$uci.set(this.configFile, section, 'hostname', this.newInterfaceName)
      this.$uci.set(this.configFile, section, 'dhcp', '1')
      this.$uci.save().then(() => {
        this.$uci.apply().then(() => {
          this.load().then(() => {
            this.updateInterfaceList()
          })
        })
      })
      this.newInterfaceName = ''
    },

    updateInterfaceList () {
      const sections = this.$uci.sections(this.configFile, this.type)
      this.sectionId = sections[sections.length - 1]['.name']
      this.sectionName = sections[sections.length - 1].name
      this.load()
      this.visible = true
    }
  }
}
</script>
