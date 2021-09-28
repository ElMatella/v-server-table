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
    <v-data-iterator
        v-else
        v-bind="$attrs"
        :items="!error ? items : []"
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
      <template #header>
        <v-progress-linear v-if="loading" indeterminate />
        <v-alert v-if="error" type="error" class="my-4">
          {{ error }}
        </v-alert>
      </template>
    </v-data-iterator>
  </div>
</template>

<script>
import debounce from 'lodash.debounce'

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
        this.page = 1
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
  }
}
</script>
