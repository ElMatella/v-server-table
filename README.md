# vuetify-server-table

This is an extension of the great vuetify datatable. It allows you to control server side pagination really easily.

## Demo
[![Edit Vuetify Playground (forked)](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/vuetify-playground-forked-y8omg?fontsize=14&hidenavigation=1&module=%2Fsrc%2Flayout.vue&theme=dark)

Or just go to https://y8omg.csb.app/ (not sure if this works, try the CodeSandbox button instead if it does not)

## Installation
```
yarn add @hammerbot/v-server-table
```

Then, use it in a Vue component:

```vue
<template>
    <v-server-table
      :fetch-fn="fetchFn"
      :headers="headers"
    />
</template>

<script>
import VServerTable from '@hammerbot/v-server-table'
export default {
  components: {
    VServerTable
  },
  data () {
    return {
      headers: [
        {
          value: 'id',
          text: 'ID'
        },
        {
          value: 'title',
          text: 'Title'
        }
      ]
    }
  },
  methods: {
    async fetchFn ({ size, page, search }) {
      // Insert your server side logic here using size, page and search parameters
      const { data } = await this.$api.get('/articles', {
        params: {
          size,
          page,
          search
        }
      })
      
      // Return an object following this structure:
      // -> hits: Contains the items of the current page
      // -> total: Contains the number of total items you are iterating over
      // -> page: The current page
      // -> size: The size fetched.
      return {
        hits: data.hits,
        total: data.total,
        page,
        size
      }
    }
  }
}
</script>
```

## Configuration

This component wraps the original [`v-data-table`](https://vuetifyjs.com/en/api/v-data-table/) component. Check out its [official documentation](https://vuetifyjs.com/en/api/v-data-table/) to find the options you are looking for.

### Pagination Options
The only added prop allows you to control more easily the pagination options:

```vue
<template>
    <v-server-table
      :fetch-fn="fetchFn"
      :headers="headers"
      :pagination-options="[10, 20, 50, 200]"
    />
</template>
```

Defaults to `[25, 50, 100, 200, 500]`

## Behaviour

The component uses lodash debounce function to trigger server side fetching only once in 500ms. The component takes care of the orchestration of the requests. You don't have to wait for a fetch to finish in order to continue navigating between pages.
