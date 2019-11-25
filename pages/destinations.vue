<template>
  <section class="destinations">
    <h1>TGVMax Finder</h1>
    <ul class="destinations">
      <li class="destination" v-for="(destination, i) in destinations" :key="i">
        <b class="destination__name">{{ destination.name }}</b>
        <ul class="destination__times">
          <li v-for="(train, j) in destination.trains" :key="j">{{ timeFormat(train.departureTime) }}</li>
        </ul>
      </li>
    </ul>
  </section>
</template>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      destinations: undefined
    }
  },

  methods: {
    timeFormat: (date) => date.toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),

    async getTrains (from, date, results = 10) {
      const url = `https://data.sncf.com/api/records/1.0/search/?dataset=tgvmax&rows=${results}&facet=destination&refine.origine=${from}&exclude.destination=${from}&refine.date=${date}/`
      const { data } = await axios.get(url)

      return data.records.map(({ fields }) => ({
        name: fields.destination,
        departureTime: new Date(`${date} ${fields.heure_depart}`),
        arrivalDate: new Date(`${date} ${fields.heure_arrivee}`),
        train: fields.train_no
      }))
    },

    groupByDestination (records) {
      const routes = records.reduce((unique, record) => {
        if (unique.map(u => u.name).includes(record.name)) {
          return unique.map(route => {
            if (route.name === record.name) {
              const newRoute = route
              newRoute.trains.push({
                id: record.train,
                departureTime: record.departureTime,
                arrivalDate: record.arrivalDate
              })
              return newRoute
            }
            return route
          })
        } else {
          return [...unique, {
            name: record.name,
            trains: [{
              id: record.train,
              departureTime: record.departureTime,
              arrivalDate: record.arrivalDate
            }]
          }]
        }
      }, [])

      const sortedRoutes = routes.map(route => {
        route.trains.sort((a, b) => b.departureTime < a.departureTime)
        return route
      }).sort((a, b) => b.trains[0].departureTime < a.trains[0].departureTime)

      return sortedRoutes
    }
  },

  mounted () {
    const from = this.$route.query.fromStation.replace(' ', '+')
    const date = this.$route.query.departureDate
    this.getTrains(from, date, 100)
      .then(res => {
        this.destinations = this.groupByDestination(res)
      })
  }
}
</script>

<style>
  .destinations {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .destination {
    display: flex;
    padding: 10px 0;
    border-bottom: 1px solid #0002;
    justify-content: space-between
  }

  .destination__times {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .destination__times li {
    display: inline-block;
    font-size: 12px;
    background-color: #eee;
    padding: 2px 5px;
    margin-right: 5px;
  }
</style>
