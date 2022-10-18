<template>
  <div>

    <a-modal
    :title="'Interface: ' +  interfaceName" v-model="isVisible" @cancel="closeModal" :width="500"
    :maskClosable = false
    >
      <vuci-form
        uci-config="task"
        @applied="closeModal"

      >
        <vuci-named-section
          type="interface"
          :name="sid"
          :label="'First Task'"
          :collapsible="false"
          v-slot="{ s }"
          :card="false"
        >
          <vuci-form-item-select
            @change='handleProtoChange'
            :uci-section="s"
            :label="'Protocol'"
            name="protocol"
            :options="protocols"
            :required="true"
          />

          <div>

            <vuci-form-item-input depend="protocol === 'static'"
            :uci-section="s"
            :label="'Address'"
            name="address"
            rules="ip4addr"
            :required="true"
            />

            <vuci-form-item-input depend="protocol === 'static'"
              :uci-section="s"
              :label="'Netmask'"
              name="netmask"
              rules="netmask4"
              :required="true"
            />
            <vuci-form-item-input depend="protocol === 'static'"
              :uci-section="s"
              :label="'Gateway'"
              name="gateway"
              rules="ip4addr"
              :required="true"
            />
            <vuci-form-item-list depend="protocol === 'static'"
              :uci-section="s"
              :label="'DNS'"
              name="dns"
              rules="ip4addr"
            />
          </div>
        </vuci-named-section>
      </vuci-form>
      <template #footer>
        <div></div>
      </template>
    </a-modal>
  </div>
</template>

<script>
export default {
  props: { sid: String, interfaceName: String, modalVisibility: Boolean },
  data () {
    return {
      isVisible: this.modalVisibility,
      protocols: [
        ['static', 'Static address'],
        ['dhcp', 'DHCP Client']
      ]
    }
  },
  methods: {

    closeModal () {
      this.isVisible = false
      this.$emit('closeInterface', this.isVisible)
    },
    handleProtoChange (self) {
      if (self.model === 'dhcp') {
        this.$uci.set('task', this.sid, 'dns', undefined)
        this.$uci.set('task', this.sid, 'netmask', undefined)
        this.$uci.set('task', this.sid, 'gateway', undefined)
        this.$uci.set('task', this.sid, 'address', undefined)
        // this line must be here for the solution to work (don't use this at home)
        this.$uci.set('task', this.sid, 'workaround', undefined)
      } else {
        this.$uci.reset()
      }
    }
  }
}
</script>
