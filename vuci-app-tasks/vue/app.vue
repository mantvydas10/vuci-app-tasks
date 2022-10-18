<template>
  <div>
      <vuci-form uci-config="task" :key="reRenderTable">
          <vuci-typed-section type="interface" :columns="columns">
              <template #name="{ s }">
                  <vuci-form-item-dummy :uci-section="s" name="name" />
              </template>
              <template #address="{ s }">
                  <vuci-form-item-dummy :uci-section="s" name="address" />
              </template>
              <template #netmask="{ s }">
                  <vuci-form-item-dummy :uci-section="s" name="netmask" />
              </template>
              <template #buttons="{s}">
                  <a-button type="primary" @click="handleEdit(s['.name'], s.name)" v-text="'Edit'"
                      style="margin-right: 5px"></a-button>
                  <a-button type="danger" @click="showDelete(s['.name'], s.name)" v-text="'Delete'"
                      style="margin-right: 5px">
                  </a-button>
              </template>
          </vuci-typed-section>
          <template #footer>
              <div style="margin: 20px">
                  <div style="display: flex">
                      <label style="margin-right: 20px; padding-top: 6px;">Interface name:
                      </label>
                      <a-input v-model="newInterfaceName" style="margin-right: 5px; width: 200px"></a-input>

                      <a-button @click="handleAddInterface" type="primary" v-text="'Create'" style="margin-left: 20px">
                      </a-button>
                  </div>
              </div>
          </template>
      </vuci-form>
      <p v-if="hasError" style="margin-top: -15px; margin-left: 137px; color:red;">{{ errorMessage }}</p>
      <div>
          <edit-modal @closeInterface="closeInterface" :modalVisibility="modalVisibility" :key="reRenderModal"
              :sid="currentSid" :interfaceName="selectedInterfaceName"></edit-modal>
      </div>
  </div>
</template>

<script>
import editModal from './components/interfaceModal.vue'
export default {
  components: { editModal },
  data () {
    return {
      errorMessage: '',
      hasError: false,
      reRenderModal: true,
      modalVisibility: false,
      newInterfaceName: '',
      selectedInterfaceName: '',
      currentSid: '',
      reRenderTable: false,

      columns: [
        { name: 'name', label: 'Interface name' },
        { name: 'address', label: 'Address' },
        { name: 'netmask', label: 'Netmask' },
        { name: 'buttons', label: '' }
      ]
    }
  },

  methods: {
    handleAddInterface () {
      if (!this.checkNameValidation(this.newInterfaceName)) {
        return
      }
      this.createInterface()
    },

    checkNameValidation (intName) {
      let validated = false
      const nameLen = intName.length
      if (nameLen === 0 || !intName) {
        this.errorMessage = 'Interface name field is required'
      } else if (nameLen > 30) {
        this.errorMessage = 'You can enter [1-30] characters only.'
      } else if (this.interfaceExists(intName)) {
        this.errorMessage = 'This interface name already exist'
      } else {
        validated = true
      }
      this.hasError = !validated
      return validated
    },

    async createInterface () {
      this.$spin()
      const sid = this.$uci.add('task', 'interface')
      await this.$uci.set('task', sid, 'name', this.newInterfaceName)
      await this.$uci.set('task', sid, 'protocol', 'static')
      await this.$uci.save()
      await this.$uci.apply('task')
      this.findLatestSid()
        .then(response => {
          this.currentSid = response
          this.selectedInterfaceName = this.newInterfaceName
          this.newInterfaceName = ''
          this.handleModalRefresh()
          this.modalVisibility = true
          this.handleTableRefresh()
          this.hasError = false
          this.errorMessage = ''
        })

      this.$spin(false)
    },
    async findLatestSid () {
      await this.$uci.load('task')
      const data = this.$uci.sections('task')
      return data[data.length - 1]['.name']
    },
    interfaceExists (intName) {
      this.$uci.load('task')
      const sections = this.$uci.sections('task')
      for (let i = 0; i < sections.length; i++) {
        if (intName === sections[i].name) {
          return true
        }
      }
      return false
    },

    closeInterface (visibility) {
      this.modalVisibility = visibility
      this.handleTableRefresh()
    },

    handleTableRefresh () {
      this.reRenderTable = !this.reRenderTable
    },

    handleModalRefresh () {
      this.reRenderModal = !this.reRenderModal
    },
    showDelete (sid, name) {
      this.$confirm({
        content: 'Are you sure you want to delete ' + name + ' interface?',
        onOk: () => {
          this.handleDelete(sid)
        }
      })
    },
    async handleDelete (sid) {
      this.$spin()
      await this.$uci.del('task', sid)
      await this.$uci.save()
      await this.$uci.apply('task')
      this.handleTableRefresh()
      this.$spin(false)
    },
    handleEdit (sid, name) {
      this.selectedInterfaceName = name
      this.currentSid = sid
      this.handleModalRefresh()
      this.modalVisibility = true
    }

  }
}
</script>
