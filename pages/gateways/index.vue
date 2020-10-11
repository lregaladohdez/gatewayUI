<template>
  <section class="section">
    <div class="column">
      <nav class="breadcrumb" aria-label="breadcrumbs">
        <ul>
          <li><a href="/">Home</a></li>
          <li><a href="/gateways">Gateways</a></li>
        </ul>
      </nav>
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
        :data="data"
        :striped="true"
        :loading="isLoading"
        label="Gateways"
        :hoverable="true"
      >
        <b-table-column v-slot="props" field="id" label="ID">{{
          props.row.id
        }}</b-table-column>
        <b-table-column v-slot="props" field="serial" label="Serial">{{
          props.row.serial
        }}</b-table-column>
        <b-table-column v-slot="props" field="name" label="Name">{{
          props.row.name
        }}</b-table-column>
        <b-table-column v-slot="props" field="address" label="Address">{{
          props.row.address
        }}</b-table-column>
        <b-table-column v-slot="props" field="peripheralsCount" label="Devices">
          <b-tag type="is-info">{{
            props.row.peripheralsCount
          }}</b-tag></b-table-column
        >
        <b-table-column v-slot="props" field="createdAt" label="CreatedAt">
          {{
            props.row.createdAt
              ? new Date(props.row.createdAt).toLocaleDateString()
              : 'unknown'
          }}
        </b-table-column>
        <b-table-column v-slot="props" label="Actions">
          <a
            class="is-primary button is-light field is-centered"
            :href="'/gateways/' + props.row.id"
            ><b-icon icon="information"></b-icon
          ></a>
          <button
            class="is-primary is-light button field is-centered"
            @click="onEdit(props.row)"
          >
            <b-icon icon="pencil"></b-icon>
          </button>
          <button
            class="is-danger is-light button field is-centered"
            @click="confirmDeleteGateway(props.row.id, props.row.name)"
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
        <Gateway
          :gateway="gateway"
          @close="isModalActive = false"
          @confirm="fetchData()"
        >
          <modal-form></modal-form>
        </Gateway>
      </b-modal>
    </div>
  </section>
</template>

<script>
import Gateway from '~/components/Gateway'
export default {
  components: {
    Gateway,
  },
  fetch() {
    this.fetchData()
  },
  data() {
    return {
      isLoading: true,
      data: [],
      isModalActive: false,
      gateway: {},
    }
  },
  async mounted() {
    await this.fetchData()
  },
  methods: {
    onEdit(gateway) {
      this.gateway = gateway
      this.isModalActive = true
    },
    onAdd() {
      this.gateway = {}
      this.isModalActive = true
    },
    async fetchData() {
      try {
        this.isModalActive = false
        const gateways = await this.$axios.$get('/gateway')
        this.data = gateways
        this.isLoading = false
      } catch (error) {
        this.$buefy.toast.open({
          duration: 5000,
          queue: true,
          message: `Can't load Gateways. ${error}`,
          position: 'is-top',
          type: 'is-danger',
        })
        this.isLoading = false
      }
    },
    confirmDeleteGateway(id, name) {
      this.$buefy.dialog.confirm({
        title: `Deleting Gateway`,
        message: `Are you shure  you want to <b>delete</b>  Gateway ${id} ${name}`,
        confirmtText: 'Delete',
        type: 'is-danger',
        hasIcon: true,
        onConfirm: async () => {
          try {
            await this.$axios.delete(`/gateway/${id}`)
            this.$buefy.toast.open({
              duration: 5000,
              queue: true,
              message: `Gateway ${id} deleted`,
              position: 'is-top',
              type: 'is-success',
            })
            await this.fetchData()
          } catch (error) {
            this.$buefy.toast.open({
              duration: 5000,
              queue: true,
              message: `Can't delete Gateway ${id}. ${error}`,
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
