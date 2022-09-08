<template>
  <section>
    <form @submit.stop.prevent="ProcessGateway()">
      <div class="modal-card" style="width: auto">
        <header class="modal-card-head">
          <p class="modal-card-title">
            {{
              myGateway.id ? `${myGateway.id} ${myGateway.name}` : 'NewGateway'
            }}
          </p>
          <button type="button" class="delete" @click="$parent.close()" />
        </header>
        <section class="modal-card-body">
          <b-field label="Serial">
            <b-input
              v-model="myGateway.serial"
              placeholder="Gateway Serial Number"
              required
            >
            </b-input>
          </b-field>
          <b-field label="Name">
            <b-input
              v-model="myGateway.name"
              placeholder="Gateway name"
              required
            >
            </b-input>
          </b-field>
          <b-field label="Address">
            <b-input
              v-model="myGateway.ipv4"
              placeholder="Gateway ipv4 address XXX.XXX.XXX.XXX"
              required
              validation-message="Enter a valid ipv4 address XXX.XXX.XXX.XXX"
              pattern="\b((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)(\.|$)){4}\b"
            >
            </b-input>
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
            {{ myGateway.id ? 'Update' : 'Add' }}
          </button>
        </footer>
      </div>
    </form>
  </section>
</template>
<script>
export default {
  props: {
    gateway: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      isLoading: false,
      myGateway: this.gateway,
    }
  },
  methods: {
    async ProcessGateway() {
      delete this.myGateway.peripherals
      let action = null
      if (this.myGateway.id) {
        action = this.$axios.patch(
          `/gateways/${this.myGateway.id}`,
          this.myGateway
        )
      } else {
        delete this.myGateway.id
        action = this.$axios.post(`/gateways/`, this.myGateway)
      }
      try {
        await action
        this.$buefy.toast.open({
          duration: 5000,
          queue: true,
          message: this.gateway.id
            ? `Gateway Edited correctly`
            : `Gateway Created correctly`,
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
          message = `${error.response.data.message}, ${error.response.data.attrNames}`
        } else if (error.response && error.response.data) {
          message = error.response.data
        }

        this.$buefy.toast.open({
          duration: 5000,
          queue: true,
          message: this.gateway.id
            ? `Can't edit Gateway. ${message}`
            : `Can't create Gateway. ${message}`,
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
