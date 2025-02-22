<script>
import CustomTable from '../../../../components/custom/table2.vue'
import OrderApi from '@/api/orders/orders_api.js'
import {ordersMehtods} from "@/state/helpers";
import Swal from "sweetalert2";
import store from "@/state/store";
import PageHeader from "../../../../components/page-header.vue";

export default {
  data() {
    return {
      title: 'Orders',
      items: [
        {
          text: 'Home',
          href: '/',
        },
        {
          text: 'Container Orders',
          active: true,
        },
      ],
      numberOfErrors: 0,
      headers: [
        {
          label: 'ORDER NUMBER',
          field: 'order_number',
          align: 'center',
          searchable: true,
        },
        {
          label: 'LOT NUMBER',
          field: 'lot_number',
          align: 'center',
          searchable: true,
        },
        {
          label: 'POSITION',
          field: 'position',
          align: 'center',
          searchable: true,
        },
        {
          label: 'QUANTITY',
          field: 'quantity',
          align: 'center',
          searchable: true,
        },
        {
          label: 'ORDER TYPE',
          field: 'type',
          align: 'center',
          searchable: true,
        },
        {
          label: 'CUSTOMER',
          field: 'customer',
          align: 'center',
          searchable: true,
        },
        {
          label: 'PAYMENT STATUS',
          field: 'payment_status',
          align: 'center',
          searchable: true,
        },
        {
          label: 'SHIPMENT STATUS',
          field: 'shipment_status',
          align: 'center',
          searchable: true,
        },
        {
          label: 'DATE',
          field: 'date',
          align: 'center',
          searchable: true,
        },
        {
          label: 'MANAGER',
          field: 'manager',
          align: 'center',
          searchable: true,
        },
        {
          label: 'ACTIONS',
          field: 'actions',
          align: 'center',
        },
      ],
      orders: [],
      orderUrl: `${process.env.VUE_APP_ORDER_URL}/container_order/list/`,
      isLoading: false,

      table: {
        url: `/container_order/list/`,
      },
      widgets_url: `${process.env.VUE_APP_ORDER_URL}/container_order/statistic/`,
      pagination: {
        perPage: 100,
      },

      getUpdate: false,


      widgets: {
        url: `${process.env.VUE_APP_ORDER_URL}/container_order/statistic/`,
        list: [
          {
            label: 'Quantity',
            field: 'quantity',
            actual_volume: 0,
            current_volume: 0
          }
        ]
      },
    };
  },
  methods: {
    ...ordersMehtods,
    async getOrders() {
      this.isLoading = true;
      let orderApi = new OrderApi();
      let data = await orderApi.getContainerOrders();
      let orders = []
      data.results.forEach(order => {
        order.order.product = order.product
        order.order.sending_type = order.sending_type
        orders.push(order.order)
      })
      this.orders = orders
      this.isLoading = false
    },
    setToUpdateOrder(order) {
      this.setCurrentlyUpdating(order)
      this.$router.push({name: 'orders_container_update', params: {id: order.order.order_number}})
    },
    deleteOrderConfirmation(order) {
      Swal.fire({
        position: "center",
        icon: "error",
        title: `You are about to delete order ${(order.order_number).toString().toUpperCase()}`,
        text: 'Deleting your order will remove all of its information from our database.',
        showDenyButton: true,
        showConfirmButton: true,
        confirmButtonText: 'Yes, Delete It',
        denyButtonText: 'Cancel',
        cancelButtonColor: 'transparent',
        focusConfirm: false,
        inputLabel: `Please type Order${(order.order_number).toString().toUpperCase()} to confirm`,
        input: 'email',
        inputPlaceholder: `Order${(order.order_number).toString().toUpperCase()}`,
        inputValidator: (value) => {
          return new Promise((resolve) => {
            if (value === 'Order' + (order.order_number).toString().toUpperCase()) {
              resolve(this.deleteOrder(order))
            } else {
              resolve('Order number did not match :)')
            }
          })
        }
      });
    },

    deleteOrder(order) {
      fetch(`${process.env.VUE_APP_ORDER_URL}/container_order/list/${order.order_number}/delete/`, {
        method: 'DELETE',
      }).then(response => {
        if (response.ok) {
          this.getUpdate = !this.getUpdate
          Swal.fire({
            position: 'center',
            icon: 'success',
            title: 'Order has been deleted successfully',
            showConfirmButton: false,
            timer: 3000
          })
        } else {
          Swal.fire({
            position: 'center',
            icon: 'error',
            title: 'An error occurred while deleting order',
            showConfirmButton: false,
          })
        }
      }).catch(error => {
        Swal.fire({
          position: 'center',
          icon: 'error',
          title: `ERROR \n ${error}`,
          showConfirmButton: true,
          confirmButtonText: 'Try Again',
        }).then((result) => {
          if (result.isConfirmed) {
            if (this.numberOfErrors >= 2) {
              Swal.fire({
                position: 'center',
                icon: 'warning',
                title: `Too Many Tries...\nPlease, talk to IT department to fix the problem`,
                showConfirmButton: false,
                timer: 10000,
                willClose: () => {
                  window.location.reload()
                }
              })
            } else {
              this.numberOfErrors += 1
              this.deleteOrderConfirmation(order)
            }
          }
        })
      })
    },

    getAccount(account) {
      return store.state.users_list.filter(user => user.id === account)[0] || {full_name: 'Unknown'}
    },

    pageChange(page) {
      this.pagination.currentPage = page
    },

    getTableData(data, field) {
      return data.row['order'][field]
    }
  },
  async mounted() {
    // await this.getOrders()
  },
  components: {
    CustomTable,
    PageHeader
  },
};
</script>

<template>
  <PageHeader :title="title" :items="items"/>
  <CustomTable
      name="ORDERS TABLE"
      id="orders_table"
      :headers="headers"
      :selectable="true"
      :searchable="true"
      :url="table.url"
      :pagination="pagination"
      :getUpdate="getUpdate"
      :widgets="widgets"
  >
    <template v-slot:top-right>
      <div class="btn-group">
        <button type="button" class="btn btn-success dropdown-toggle" data-bs-toggle="dropdown" aria-expanded="false">
          Create order
        </button>
        <div class="dropdown-menu dropdownmenu-success">
          <router-link class="dropdown-item" :to="{name: 'create_container'}">Container</router-link>
          <router-link class="dropdown-item" :to="{name: 'create_wagon'}">Wagon</router-link>
          <router-link class="dropdown-item" :to="{name: 'create_empty_wagon'}">Empty wagon</router-link>
        </div>
      </div>
    </template>

    <template v-slot:quantity="slotProps">
      <span class="badge" :class="{
        'badge-outline-success' : slotProps.row.order.filled_quantity === slotProps.row.order.quantity,
        'badge-outline-danger' : slotProps.row.order.filled_quantity < slotProps.row.order.quantity,
      }">
        {{ slotProps.row.order.filled_quantity + '/' + slotProps.row.order.quantity }}
      </span>
    </template>

    <template v-slot:order_number="slotProps">

      <span class="badge badge-soft-secondary fs-12">
        <router-link :to="{name: 'orders_container_detail', params: {id: getTableData(slotProps, 'order_number')}}">
          {{ getTableData(slotProps, 'order_number') }}
        </router-link>
      </span>
    </template>

    <template v-slot:actions="slotProps">

      <font-awesome-icon @click="setToUpdateOrder(slotProps.row)" icon="fa-solid fa-pen-to-square"
                         class="c_icon mx-2 c_icon_hoverable"/>

      <font-awesome-icon @click="deleteOrderConfirmation(slotProps.row.order)"
                         icon="fa-solid fa-trash" class="c_icon c_icon_hoverable text-danger"/>
    </template>

    <template v-slot:customer="slotProps">
      <div>
        <span class="rounded-circle bg-soft-secondary text-secondary mx-1 px-2">
          {{ getAccount(getTableData(slotProps, 'customer'))['full_name'][0].toUpperCase() }}
        </span>
        <span>
          {{ getAccount(getTableData(slotProps, 'customer'))['full_name'] }}
        </span>
      </div>
    </template>

    <template v-slot:manager="slotProps">
      <div>
        <span class="rounded-circle bg-soft-secondary text-secondary mx-1 px-2">
          {{ getAccount(getTableData(slotProps, 'manager'))['full_name'][0].toUpperCase() }}
        </span>
        <span>
          {{ getAccount(getTableData(slotProps, 'manager'))['full_name'] }}
        </span>
      </div>
    </template>

    <template v-slot:type="slotProps">
      <span v-if="getTableData(slotProps, 'type') == 'Export'" class="badge bg-success">
        Export
      </span>
      <span v-else-if="getTableData(slotProps, 'type') == 'Import'" class="badge bg-primary">
        Import
      </span>
      <span v-else-if="getTableData(slotProps, 'type') == 'Transit'" class="badge bg-warning">
        Transit
      </span>
    </template>

    <template v-slot:shipment_status="slotProps">
      <span v-if="getTableData(slotProps, 'shipment_status') == 'Completed'" class="badge badge-outline-success">
        Completed
      </span>
      <span v-else-if="getTableData(slotProps, 'shipment_status') == 'Delivered'"
            class="badge badge-outline-primary">
        Delivered
      </span>
      <span v-else-if="getTableData(slotProps, 'shipment_status') == 'In process'"
            class="badge badge-outline-warning">
        In process
      </span>
    </template>

    <template v-slot:payment_status="slotProps">
      <span v-if="getTableData(slotProps, 'payment_status') == 'Reserved'" class="badge badge-outline-success">
        Reserved
      </span>
      <span v-else-if="getTableData(slotProps, 'payment_status') == 'Received'" class="badge badge-outline-primary">
        Received
      </span>
      <span v-else-if="getTableData(slotProps, 'payment_status') == 'Issued'" class="badge badge-outline-warning">
        Issued
      </span>
    </template>

    <template v-slot:lot_number="slotProps">
      {{
        getTableData(slotProps, 'lot_number') === null || getTableData(slotProps, 'lot_number') === '' ? '-'
            : getTableData(slotProps, 'lot_number')
      }}
    </template>

  </CustomTable>
</template>