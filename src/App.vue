<template>
  <div id="app">
    <b-container>
      <b-row>
        <b-col>
          <h1>Клиент для Star Wars API</h1>
        </b-col>
      </b-row>
      <b-row>
        <b-col cols="3">
          <b-input-group style="margin-bottom: 15px">
            <b-form-input v-model="query" type="text" placeholder="Введите запрос" @keyup.enter.native="getData"></b-form-input>
            <b-input-group-append>
              <b-btn @click="getData" variant="outline-success">Искать</b-btn>
            </b-input-group-append>
          </b-input-group>
          <b-card>
            <b-list-group v-if="searchHistory">
              <b-list-group-item v-for="item in searchHistory">
                  {{item.date}}: <b>{{item.query}}</b>
              </b-list-group-item>
            </b-list-group>
          </b-card>
        </b-col>
        <b-col cols="9">
          <b-card v-if="starships">
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
                 <!-- <b-row class="mb-2" v-for="(value, key) in row.item">-->
                  <b-row class="mb-2" v-for="(header, key) in headers">
                    <b-col sm="3" class="text-sm-right"><b>{{header}}:</b></b-col>
                    <b-col>{{row.item[key]}}</b-col>
                  </b-row>
                  <b-button size="sm" @click="row.toggleDetails">Скрыть</b-button>
                </b-card>
              </template>
            </b-table>
            <b-pagination v-if="total>10" size="md" :total-rows="total" v-model="currentPage" :per-page="10" @change="getPageData">
            </b-pagination>
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
      fields: {
        created: {label: 'Дата создания', sortable: true},
        name: {label: 'Наименование', sortable: true},
        starship_class: {label: 'Класс', sortable: true},
        passengers: {label: 'Вместимость (пасс.)', sortable: true},
        cost_in_credits: {label: 'Стоимость (кр.)', sortable: true},
        show_details: {label: ''}
      },
      currentPage: 1,
      response: {count: 0, starships:[]},
      searchHistory: []
    }
  },
  computed: {
    total() {
      return this.response.count
    },
    starships() {
      return this.response.results
    },
    /*searchHistory() {
    LOCAL STORAGE не реактивен
      let story_queries = localStorage.getItem('swapi_client_history_queries').split(',')
      let story_dates = localStorage.getItem('swapi_client_history_dates').split(',')
      let story = []
      for (let i = 0; i < story_queries.length; i++) {
        if (story_queries[i] !== '') { story.push({date: story_dates[i], query: story_queries[i]}) }
      }
      return story
      },*/
    headers() {
      return {
        name: 'Наименование',
        model: 'Модель',
        manufacturer: 'Производитель',
        cost_in_credits: 'Цена в кредитах',
        length: 'Длина',
        max_atmosphering_speed: 'Макс. скорость в атмосфере',
        crew: 'Персонал',
        passengers: 'Пассажировместимость',
        cargo_capacity: 'Грузоподъемность',
        consumables: 'Автономность',
        hyperdrive_rating: 'Гипердвигатель',
        MGLT: 'Скорость (MGLT)',
        starship_class: 'Класс',
        created: 'Создано',
        edited: 'Редактировано'
      }
    }

  },
  methods: {
    getData: function () {
      let url = this.query ? 'https://swapi.co/api/starships/' + '?search=' + this.query : 'https://swapi.co/api/starships/'
      this.currentPage = 1
      let xhr = new XMLHttpRequest()
      let response = ''
      xhr.open('GET', url, false)
      xhr.send()
      if (xhr.status != 200) {
        alert( xhr.status + ': ' + xhr.statusText )
      } else {
        this.response = JSON.parse(xhr.responseText)
        let storage = localStorage.getItem('swapi_client_history')
        console.log(storage)
        let story_queries = localStorage.getItem('swapi_client_history_queries') ? localStorage.getItem('swapi_client_history_queries') : ''
        let story_dates = localStorage.getItem('swapi_client_history_dates') ? localStorage.getItem('swapi_client_history_dates') : ''
        let date = new Date().toLocaleString("ru",{year: 'numeric', month: 'numeric', day: 'numeric'})
        story_queries = story_queries + ',' + this.query
        story_dates = story_dates + ',' + date
        localStorage.setItem('swapi_client_history_queries', story_queries)
        localStorage.setItem('swapi_client_history_dates', story_dates)
      }
    },
    getPageData() {
      setTimeout(()=>{
        let url = this.query ? 'https://swapi.co/api/starships/' + '?search=' + this.query : 'https://swapi.co/api/starships/'
        if (this.currentPage !== 1 && this.query === '') {
          url = this.query ? url + '&page=' + this.currentPage : url + '?page=' + this.currentPage
        }
        let xhr = new XMLHttpRequest()
        xhr.open('GET', url, false)
        xhr.send()
        if (xhr.status != 200) {
          alert( xhr.status + ': ' + xhr.statusText )
        } else {
          this.response = JSON.parse(xhr.responseText)
        }
      },200)
    },
    getHistory() {
      let story_queries = localStorage.getItem('swapi_client_history_queries').split(',')
      let story_dates = localStorage.getItem('swapi_client_history_dates').split(',')
      let story = []
      for (let i = story_queries.length - 1; i > - 1; i--) {
        if (story_queries[i] !== '') { story.push({date: story_dates[i], query: story_queries[i]}) }
      }
      story = story.slice(0, 10)
      return story
    }
  },
  beforeMount() {
    this.searchHistory = this.getHistory()
  },
  beforeUpdate() {
    this.searchHistory = this.getHistory()
  }
}
</script>

<style lang="scss">
#app {
  margin-top: 15px;
}
</style>
