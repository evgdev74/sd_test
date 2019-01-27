<template>
  <div id="app">
    <b-container>
      <b-row>
        <b-col>
          <h1>Клиент для Star Wars API</h1>
        </b-col>
      </b-row>
      <b-row>
        <b-col cols="4">
          <b-card>
            <b-input-group>
              <b-form-input v-model="query" type="text" placeholder="Введите запрос"></b-form-input>
              <b-input-group-append>
                <b-btn @click="getData" variant="outline-success">Искать</b-btn>
              </b-input-group-append>
            </b-input-group>
          </b-card>
        </b-col>
        <b-col cols="8">
          <b-card>
            <b-table striped hover :items="starships" :fields="fields">
              <template slot="created" slot-scope="data">
                {{new Date(Date.parse(data.item.created)).toLocaleString("ru",{year: 'numeric', month: 'long', day: 'numeric'})}}
              </template>
              <template slot="show_details" slot-scope="row">
                <b-button size="sm" @click.stop="row.toggleDetails" class="mr-2">
                  {{ row.detailsShowing ? 'Скрыть' : 'Показать'}}
                </b-button>
              </template>
              <template slot="row-details" slot-scope="row">
                <b-card>
                  <b-row class="mb-2" v-for="(value, key) in row.item">
                    <b-col sm="3" class="text-sm-right"><b>{{key}}:</b></b-col>
                    <b-col>{{value}}</b-col>
                  </b-row>
                  <b-button size="sm" @click="row.toggleDetails">Скрыть</b-button>
                </b-card>
              </template>
            </b-table>
          </b-card>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      query: '',
      starships : [],
      fields: {
        created: {label: 'Дата создания', sortable: true},
        name: {label: 'Наименование', sortable: true},
        starship_class: {label: 'Класс', sortable: true},
        passengers: {label: 'Вместимость (пасс.)', sortable: true},
        cost_in_credits: {label: 'Стоимость (кр.)', sortable: true},
        show_details: {label: ''}
      }
    }
  },
  computed: {
    starships_data () {
      return [1, 2, 3]
    }
  },
  methods: {
    getData: function (event) {
      let url = this.query ? 'https://swapi.co/api/starships/' + '?search=' + this.query : 'https://swapi.co/api/starships/'
      let xhr = new XMLHttpRequest()
      let response = ''
      xhr.open('GET', url, false)
      xhr.send()
      if (xhr.status != 200) {
        alert( xhr.status + ': ' + xhr.statusText )
      } else {
        response = JSON.parse(xhr.responseText)
        console.dir( response )
        this.starships = response.results
      }
    }
  }

}
</script>

<style lang="scss">
#app {
  margin-top: 15px;
}


</style>
