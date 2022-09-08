<template>
  <section class="section">
    <dir class="column">
      <nav class="breadcrumb" aria-label="breadcrumbs">
        <ul>
          <li><a href="/">Home</a></li>
          <li><a href="/gateways">Gateways</a></li>
          <li>
            <a :href="'/gateways/' + id"
              >{{ id }} {{ gateway ? gateway.name : '' }}</a
            >
          </li>
        </ul>
      </nav>
    </dir>
    <div class="column">
      <div v-if="gateway" class="card">
        <div class="card-content">
          <div class="media">
            <div class="media-content">
              <p class="title is-4">{{ gateway.name }}</p>
              <p class="subtitle is-6">{{ gateway.serial }}</p>
            </div>
          </div>

          <div class="content">
            <b-field grouped group-multiline>
              <div class="control">
                <b-taglist attached>
                  <b-tag type="text">address</b-tag>
                  <b-tag type="is-info">{{ gateway.ipv4 }}</b-tag>
                </b-taglist>
              </div>
            </b-field>
          </div>
        </div>
      </div>
    </div>
    <div class="column">
      <button class="is-primary button field is-centered" @click="onAdd()">
        <span>Add</span>
        <b-icon icon="plus"></b-icon>
      </button>
      <button class="is-primary button field is-centered" @click="fetchData()">
        <span>Refresh</span>
        <b-icon icon="refresh"></b-icon>
      </button>
      <b-table
        :data="devices"
        :striped="true"
        :loading="isLoading"
        label="Peripherlas"
        :hoverable="true"
        empty="No Peripherlas attached"
      >
        <b-table-column v-slot="props" field="uuid" label="UUID">{{
          props.row.uuid
        }}</b-table-column>
        <b-table-column v-slot="props" field="vendor" label="Vendor">{{
          props.row.vendor
        }}</b-table-column>
        <b-table-column v-slot="props" field="status" label="Status">
          <button
            v-show="props.row.status == 'online'"
            class="button is-success is-light"
            @click="changePerihperalState(props.row.id, props.row.status)"
          >
            <b-icon icon="toggle-switch"></b-icon>
            <span>{{ props.row.status }}</span>
          </button>
          <button
            v-show="props.row.status !== 'online'"
            class="button is-light is-danger"
            @click="changePerihperalState(props.row.id, props.row.status)"
          >
            <b-icon icon="toggle-switch-off"></b-icon>
            <span>{{ props.row.status }}</span>
          </button>
        </b-table-column>
        <b-table-column v-slot="props" field="createdAt" label="CreatedAt">
          {{
            props.row.date
              ? new Date(props.row.date).toLocaleDateString()
              : 'unknown'
          }}
        </b-table-column>
        <b-table-column v-slot="props" label="Actions">
          <button
            class="is-primary is-light button field is-centered"
            @click="onEdit(props.row)"
          >
            <b-icon icon="pencil"></b-icon>
          </button>
          <button
            class="is-danger is-light button field is-centered"
            @click="confirmDeletePeripheral(props.row.id)"
          >
            <b-icon icon="delete"></b-icon>
          </button>
        </b-table-column>
      </b-table>
      <b-modal
        v-if="isModalActive"
        v-model="isModalActive"
        has-modal-card
        trap-focus
        :destroy-on-hide="false"
        aria-role="dialog"
        aria-modal
      >
        <Peripheral
          :peripheral="peripheral"
          :gateway="gateway.id"
          @close="isModalActive = false"
          @confirm="fetchData()"
        >
          <modal-form></modal-form>
        </Peripheral>
      </b-modal>
    </div>
  </section>
</template>

<script>
import Peripheral from '~/components/Peripheral'
export default {
  components: {
    Peripheral,
  },
  data() {
    return {
      isLoading: true,
      isModalActive: false,
      gateway: null,
      devices: [],
      id: this.$route.params.id,
    }
  },
  async mounted() {
    await this.fetchData()
  },
  methods: {
    onEdit(peripheral) {
      this.peripheral = peripheral
      this.isModalActive = true
    },
    onAdd() {
      this.peripheral = {}
      this.isModalActive = true
    },
    async fetchData() {
      this.isModalActive = false
      try {
        ;[this.gateway, { data: this.devices }] = await Promise.all([
          this.$axios.$get(`/gateways/${this.id}`),
          await this.$axios.$get(`/peripherals?gatewayId=${this.id}`),
        ])
      } catch (error) {
        this.$buefy.toast.open({
          duration: 5000,
          queue: true,
          message: `Can't load Gateway ${this.id}. Erorr ${error}`,
          position: 'is-top',
          class: 'is-danger',
        })
      } finally {
        this.isLoading = false
      }
    },
    async changePerihperalState(id, statusOld) {
      try {
        const status = statusOld === 'online' ? 'offline' : 'online'
        this.isLoading = true
        await this.$axios.patch(`/peripherals/${id}`, {
          status,
        })
        await this.fetchData()
      } catch (error) {
        this.$buefy.toast.open({
          duration: 5000,
          queue: true,
          message: `Can't change status of Pheripheral ${id}. Erorr ${error}`,
          position: 'is-top',
          class: 'is-danger',
        })
      } finally {
        this.isLoading = false
      }
    },
    confirmDeletePeripheral(id) {
      this.$buefy.dialog.confirm({
        title: `Deleting Peripheral`,
        message: `Are you shure  you want to <b>delete</b>  Peripheral ${id}`,
        confirmtText: 'Delete',
        type: 'is-danger',
        hasIcon: true,
        onConfirm: async () => {
          try {
            await this.$axios.delete(`/peripherals/${id}`)
            this.$buefy.toast.open({
              duration: 5000,
              queue: true,
              message: `Peripheral ${id} deleted`,
              position: 'is-top',
              type: 'is-success',
            })
            await this.fetchData()
          } catch (error) {
            this.$buefy.toast.open({
              duration: 5000,
              queue: true,
              message: `Can't delete Peripheral ${id}. ${error}`,
              position: 'is-top',
              type: 'is-danger',
            })
          }
        },
      })
    },
  },
}
</script>
