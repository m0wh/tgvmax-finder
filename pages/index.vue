<template>
  <section class="index">
    <h1>TGVMax Finder</h1>
    <p>Trouvez un endroit où aller avec TGVMax dès aujourd'hui.</p>
    <form>
      <div class="form-group">
        <label for="departure-date">Date de départ</label>
        <input type="date" name="departure_date" id="departure-date" v-model="departureDate">
      </div>

      <div class="form-group">
        <label for="from-station">Gare de départ</label>
        <input type="text" placeholder="Ex. : Lyon (gares intramuros)" v-model="fromStation">
      </div>

      <nuxt-link :to="{ path: 'destinations', query: { departureDate, fromStation }}">Rechercher</nuxt-link>
    </form>
  </section>
</template>

<script>
export default {
  data () {
    return {
      departureDate: '',
      fromStation: ''
    }
  }
}
</script>
