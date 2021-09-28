<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
    >
    </v-app-bar>
    <v-main>
      <div class="pa-10">
        <div class="d-flex">
          <v-spacer />
          <v-text-field
            label="Search"
            v-model="search"
          ></v-text-field>
        </div>
        <v-tabs>
          <v-tab>
            Tables
          </v-tab>
          <v-tab>
            Iterators
          </v-tab>
          <v-tab-item>
            <server-table
                :headers="headers"
                :search="search"
                :fetch-fn="fetch1"
                dense
                class="mb-4"
            >
              <template #item.id="{ item }">
                {{ item.id }} Templated
              </template>
            </server-table>
            <server-table
                :headers="headers"
                :fetch-fn="fetchErrorOnFirstPage"
                dense
                class="mb-4"
            >
              <template #item.id="{ item }">
                {{ item.id }} Templated
              </template>
            </server-table>
            <server-table
                :headers="headers"
                :fetch-fn="fetchErrorOnSecondPage"
                dense
            >
              <template #item.id="{ item }">
                {{ item.id }} Templated
              </template>
            </server-table>
          </v-tab-item>
          <v-tab-item>
            <server-iterator
                :headers="headers"
                :search="search"
                :fetch-fn="fetch1"
                dense
                class="mb-4"
            >
              <template #item="{ item }">
                <v-card>
                  {{ item.id }} Templated
                </v-card>
              </template>
            </server-iterator>
            <server-iterator
                :headers="headers"
                :fetch-fn="fetchErrorOnFirstPage"
                dense
                class="mb-4"
            >
              <template #item="{ item }">
                {{ item.id }} Templated
              </template>
            </server-iterator>
            <server-iterator
                :headers="headers"
                :fetch-fn="fetchErrorOnSecondPage"
                dense
            >
              <template #item="{ item }">
                <v-card>
                  {{ item.id }} Templated
                </v-card>
              </template>
            </server-iterator>
          </v-tab-item>
        </v-tabs>

      </div>
    </v-main>
  </v-app>
</template>

<script>
import {VServerTable, VServerIterator} from "@hammerbot/v-server-table";
import _ from 'lodash'

const fetch1 = async ({ page, size }) => {
  await new Promise(resolve => setTimeout(resolve, 2000))
  const result = {
    hits: _.range(size).map(i => {
      return {
        id: (page - 1) * size + i + 1,
        name: `page ${page}: ${i + 1}`
      }
    }),
    page,
    total: 1000,
    size
  }
  return result
}

const fetchErrorOnSecondPage = async ({ page, size }) => {
  await new Promise(resolve => setTimeout(resolve, 2000))
  if (page === 2) {
    throw new Error('Some error has happened.')
  }
  const result = {
    hits: _.range(size).map(i => {
      return {
        id: (page - 1) * size + i + 1,
        name: `page ${page}: ${i + 1}`
      }
    }),
    page,
    total: 1000,
    size
  }
  return result
}

const fetchErrorOnFirstPage = async ({ page, size }) => {
  await new Promise(resolve => setTimeout(resolve, 2000))
  if (page === 1) {
    throw new Error('Some error has happened.')
  }
  const result = {
    hits: _.range(size).map(i => {
      return {
        id: (page - 1) * size + i + 1,
        name: `page ${page}: ${i + 1}`
      }
    }),
    page,
    total: 1000,
    size
  }
  return result
}

export default {
  components: {
    ServerTable: VServerTable,
    ServerIterator: VServerIterator
  },
  name: 'App',
  data () {
    return {
      headers: [
        {
          value: 'id',
          text: 'ID'
        },
        {
          value: 'name',
          text: 'Name'
        }
      ],
      fetch1,
      fetchErrorOnSecondPage,
      fetchErrorOnFirstPage,
      search: null
    }
  }
};
</script>
