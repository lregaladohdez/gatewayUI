<template>
  <section>
    <form @submit.stop.prevent="ProcessPeripheral()">
      <div class="modal-card" style="width: auto">
        <header class="modal-card-head">
          <p class="modal-card-title">
            {{
              myPeripheral.id
                ? `${myPeripheral.id} ${myPeripheral.vendor}`
                : 'New Peripheral'
            }}
          </p>
          <button type="button" class="delete" @click="$parent.close()" />
        </header>
        <section class="modal-card-body">
          <b-field label="UUID">
            <b-input
              v-model="myPeripheral.uuid"
              placeholder="Peripheral UUID"
              type="text"
              validation-message="Enter a valid uuid"
              required
            >
            </b-input>
          </b-field>
          <b-field label="Vendor">
            <b-input
              v-model="myPeripheral.vendor"
              placeholder="Peripheral Vendor Name"
              required
            >
            </b-input>
          </b-field>
          <b-field label="Status">
            <b-switch
              v-model="status"
              passive-type="is-danger"
              type="is-success"
            >
              {{ status ? 'online' : 'offline' }}
            </b-switch>
          </b-field>
        </section>
        <footer class="modal-card-foot">
          <button
            class="button"
            type="button"
            :enabled="!isLoading"
            @click="$parent.close()"
          >
            Close
          </button>
          <button class="button is-primary" :enabled="!isLoading" type="submit">
            {{ myPeripheral.id ? 'Update' : 'Add' }}
          </button>
        </footer>
      </div>
    </form>
  </section>
</template>
<script>
export default {
  props: {
    peripheral: {
      type: Object,
      required: true,
    },
    gateway: {
      type: Number,
      required: true,
    },
  },
  data() {
    return {
      isLoading: false,
      myPeripheral: this.peripheral,
      status: this.peripheral.status === 'online',
    }
  },
  methods: {
    async ProcessPeripheral() {
      let action = null
      this.myPeripheral.status = this.status ? 'online' : 'offline'
      this.myPeripheral.gatewayId = this.gateway
      if (this.myPeripheral.id) {
        action = this.$axios.patch(
          `/peripherals/${this.myPeripheral.id}`,
          this.myPeripheral
        )
      } else {
        delete this.myPeripheral.id
        action = this.$axios.post(`/peripherals/`, this.myPeripheral)
      }
      try {
        await action
        this.$buefy.toast.open({
          duration: 5000,
          queue: true,
          message: this.peripheral.id
            ? `Peripheral Edited correctly`
            : `Peripheral Created correctly`,
          position: 'is-top',
          type: 'is-success',
        })
        this.$emit('confirm')
      } catch (error) {
        let message = error.toString()
        if (
          error.response &&
          error.response.data &&
          error.response.data.details
        ) {
          message = error.response.data.details
        } else if (
          error.response &&
          error.response.data &&
          error.response.data.message
        ) {
          message = `${error.response.data.message}`
        } else if (error.response && error.response.data) {
          message = error.response.data
        }

        this.$buefy.toast.open({
          duration: 5000,
          queue: true,
          message: this.peripheral.id
            ? `Can't edit Peripheral. ${message}`
            : `Can't create Peripheral. ${message}`,
          position: 'is-top',
          type: 'is-danger',
        })
      } finally {
        this.isLoading = false
      }
    },
  },
}
</script>
