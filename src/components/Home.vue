<template>
  <v-container>
      <v-dialog
          v-model="dialog"
          width="500"
      >
          <v-card>
              <v-card-title class="text-h5 grey lighten-2">
                    {{packageName}}
              </v-card-title>

              <v-card-text>
                  <v-chip
                      class="ma-2"
                      v-for="(value, name) in tags"
                      :key="value+name"
                  >
                      {{name}}: {{value}}
                  </v-chip>
                  <v-data-table
                      :headers="infoHeaders"
                      :items="version"
                      class="elevation-1 mt-3"
                  >
                      <template v-slot:header.name="{ header }">
                          {{ header.text.toUpperCase() }}
                      </template>
                  </v-data-table>
              </v-card-text>

              <v-divider></v-divider>

              <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn
                      color="primary"
                      text
                      @click="dialog = false"
                  >
                      Close
                  </v-btn>
              </v-card-actions>
          </v-card>
      </v-dialog>
    <v-row>
        <v-col>
            <template>
                <v-card>
                    <v-card-title>
                        <v-text-field
                            v-model="search"
                            append-icon="mdi-magnify"
                            label="Search"
                            single-line
                            hide-details
                        ></v-text-field>
                    </v-card-title>
                    <v-data-table
                        :headers="headers"
                        :items="packageItems"
                        :search="search"
                    >
                        <template v-slot:item.actions="{ item }">
                            <v-icon
                                small
                                class="mr-2"
                                @click="openInfo(item)"
                            >
                                mdi-eye
                            </v-icon>
                        </template>
                    </v-data-table>
                </v-card>
            </template>
        </v-col>
    </v-row>
      <v-snackbar
          v-model="alert"
          :multi-line="true"
          absalute
          top
          right
          color="red"
      >
          No data found

          <template v-slot:action="{ attrs }">
              <v-btn
                  color="while"
                  text
                  v-bind="attrs"
                  @click="alert = false"
              >
                  <v-icon
                      small
                      class="mr-2"
                      @click="openInfo(item)"
                  >
                      mdi-close
                  </v-icon>
              </v-btn>
          </template>
      </v-snackbar>
  </v-container>
</template>

<script>
import axios from 'axios'
export default {
  name: 'Home',
  data () {
    return {
      alert: false,
      dialog: false,
      tags: null,
      search: '',
      version: [],
      packageName: '',
      headers: [
        { text: 'Name', align: 'start', filterable: true, value: 'name' },
        { text: 'Type', value: 'type' },
        { text: 'Hits', value: 'hits' },
        { text: 'Actions', value: 'actions', sortable: false }
      ],
      infoHeaders: [
        { text: 'N', align: 'start', filterable: true, value: 'n' },
        { text: 'Version', sortable: false, value: 'version' }
      ],
      packageItems: [
        {
          name: null,
          type: null,
          hits: null
        }
      ]
    }
  },
  mounted () {
    axios.get('https://data.jsdelivr.com/v1/stats/packages')
      .then(response => {
        this.packageItems = response.data
      })
      .catch(e => {
        console.log(e)
      })
  },
  methods: {
    openInfo (item) {
      axios.get(`https://data.jsdelivr.com/v1/package/${item.type}/${item.name}`)
        .then(response => {
          this.tags = response.data.tags
          this.version = response.data.versions.map((item, idx) => ({ n: idx + 1, version: item }))
          this.packageName = item.name
          this.dialog = true
        })
        .catch(e => {
          this.alert = true
        })
    }
  }
}
</script>
