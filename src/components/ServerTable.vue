<template>
  <div>
    <v-skeleton-loader
        v-if="items === null && !error"
        type="table"
        class="mx-auto"
    />
    <v-alert v-else-if="items === null && error" type="error" class="my-4">
      {{ error }}
    </v-alert>
    <v-data-table
        v-else
        v-bind="$attrs"
        :items="items"
        :items-per-page="size"
        :page="page"
        :loading="loading"
        :server-items-length="total"
        :footer-props="{
          itemsPerPageOptions: paginationOptions
        }"
        :loader-height="2"
        v-on="$listeners"
        @update:items-per-page="size = $event"
        @update:page="page = $event"
    >
      <template v-for="(_, slotName) in $slots" #[slotName]>
        <slot :name="slotName" />
      </template>
      <template v-for="(_, scopedSlotName) in $scopedSlots" #[scopedSlotName]="slotData">
        <slot :name="scopedSlotName" v-bind="slotData" />
      </template>
      <template v-if="error" v-slot:body="{}">
        <tr>
          <td :colspan="$attrs.headers.length">
            <v-alert type="error" class="my-4">
              {{ error }}
            </v-alert>
          </td>
        </tr>
      </template>
    </v-data-table>
  </div>
</template>

<script>
import debounce from 'lodash.debounce'
// import {VSkeletonLoader, VDataTable, VAlert, VSpacer, VBtn, VIcon} from 'vuetify/lib'

export default {
  props: {
    fetchFn: {
      required: true,
      type: Function
    },
    search: {
      required: false,
      default: undefined,
      type: String
    },
    paginationOptions: {
      required: false,
      default: () => [25, 50, 100, 200, 500],
      type: Array
    }
  },
  data () {
    let initialSize = 10
    if (this.paginationOptions && this.paginationOptions.length) {
      initialSize = this.paginationOptions[0]
    }
    return {
      page: 1,
      size: initialSize,
      total: null,
      items: null,
      loading: false,
      pendingRequest: null,
      executingRequest: null,
      debounceFn: debounce(this.executeQueue, 500),
      error: null
    }
  },
  watch: {
    page: {
      immediate: true,
      handler () {
        this.fetch()
      }
    },
    size: {
      handler () {
        this.fetch()
      }
    },
    search: {
      handler () {
        this.fetch()
      }
    }
  },
  methods: {
    fetch () {
      // We create a request, and assign it to
      // a property in the component to use it
      // later
      this.pendingRequest = async () => {
        this.loading = true
        this.error = null
        try {
          const { hits, total } = await this.fetchFn({
            page: this.page,
            size: this.size,
            search: this.search
          })
          this.items = hits
          this.total = total
        } catch (error) {
          this.error = error
        }
        this.loading = false
      }

      // If the queue is not already running,
      // We run the queue.
      if (!this.executingRequest) {
        this.debounceFn()
      }
    },
    async executeQueue () {
      // If there is a request to run,
      // We run it
      if (this.pendingRequest) {
        // We switch from pending request to executing request
        // and reset pending request to null
        // to allow the variable to be filled again
        this.executingRequest = this.pendingRequest
        this.pendingRequest = null
        // Execute the request
        await this.executingRequest()
        // Set executing request to null
        this.executingRequest = null
        // Execute the queue again to check
        // If there is another request to
        // execute.
        await this.debounceFn()
      }
    }
  },
  /**
  render (h) {
    if (this.items === null && !this.error) {
      return h(VSkeletonLoader, {
        props: {
          type: 'table'
        }
      })
    }

    const getScopedSlots = () => {
      if (!this.error) {
        return this.$scopedSlots
      }

      return {
        'body': () => {
          const errorMessage = this.error && this.error.message ? this.error.message : this.error
          const alert = h(VAlert, {
            props: {
              type: 'error'
            },
            class: 'my-4',
          }, [errorMessage, h(VSpacer), h('template', {
            slot: 'append'
          }, [
            h(VBtn, {
              props: {
                icon: true
              },
              class: 'ml-2',
              on: {
                click: () => {
                  this.fetch()
                }
              }
            }, [
              h(VIcon, ['refresh'])
            ])
          ])])

          return h('tr', [
              h('td', {
                attrs: {
                  colspan: this.$attrs.headers.length
                }
              }, [
                  h('div', {
                    class: 'd-flex justify-center',
                  }, [alert])
              ])
          ])
        }
      }
    }

    // https://stackoverflow.com/questions/50891858/vue-how-to-pass-down-slots-inside-wrapper-component
    const children = Object.keys(this.$slots).map(slot => h('template', { slot }, this.$slots[slot]))

    return h(VDataTable, {
      attrs: {
        ...this.$attrs
      },
      props: {
        items: this.items || [],
        itemsPerPage: this.size,
        page: this.page,
        loading: this.loading,
        serverItemsLength: this.total,
        footerProps: {
          itemsPerPageOptions: this.paginationOptions
        },
        loaderHeight: 2
      },
      on: {
        ...this.$listeners,
        'update:items-per-page': (e) => {
          this.size = e
        },
        'update:page': (e) => {
          this.page = e
        }
      },
      scopedSlots: {
        ...getScopedSlots()
      },
    }, children)
  }
  */
}
</script>
