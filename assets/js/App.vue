<template>
    <v-container>
        <h1>Regarderou</h1>

        <v-row>
            <v-col>
                <v-autocomplete
                    @update:search="search($event)"
                    :items="results.results"
                    class="mx-4"
                    label="search"
                    :loading="loading"
                    clearable
                    hide-selected
                    hide-no-data
                ></v-autocomplete>
            </v-col>
        </v-row>

        <v-row justify="space-around">
            <v-card v-if="providers && results.results" width="1000">
                <v-img
                    height="500px"
                    :src="providers && results.results ? 'https://www.themoviedb.org/t/p/original/' + results.results[0].backdrop_path : 'https://cdn.pixabay.com/photo/2020/07/12/07/47/bee-5396362_1280.jpg'"
                >

                    <v-card-title class="white--text mt-8 d-flex align-center">
                        <v-avatar size="56">
                            <img
                                alt="user"
                                :src="providers && results.results ? 'https://www.themoviedb.org/t/p/original/' + results.results[0].poster_path : 'https://cdn.pixabay.com/photo/2020/06/24/19/12/cabbage-5337431_1280.jpg'"
                            >
                        </v-avatar>
                        <p class="ml-3">
                            {{ providers && results.results ? results.results[0].original_title : "Rien" }}
                        </p>
                    </v-card-title>
                </v-img>

                <v-card-text>
                    <div v-if="providers && providers.results && providers.results.FR" v-for="provider in providers.results.FR.flatrate">
                        <h2>{{ provider.provider_name }}</h2>
                        <img :src="'https://www.themoviedb.org/t/p/original/' + provider.logo_path" :alt="provider.provider_name" class="rounded-xl">
                        <div> ({{ results.results[0].release_date }})</div>
                    </div>
                    <div v-if="providers && providers.results && providers.results.FR && !providers.results.FR.flatrate">Pas de résultats.</div>
                </v-card-text>
            </v-card>
        </v-row>
    </v-container>

</template>


<script>
import axios from "axios";

export default {
    name: 'App',
    data() {
        return {
            loading: false,
            providers: {},
            movie: {},
            movieId: null,
            input: null,
            results: [],
            searchTimeout: null
        }
    },
    methods: {
        search(value) {
            if (this.searchTimeout !== null) {
                clearTimeout(this.searchTimeout);
            }

            if (value != null && value !== '') {
                this.loading = true;

                this.searchTimeout = setTimeout(
                    () => axios.get('https://api.themoviedb.org/3/search/movie?api_key=f904a8044e5207e6d815f07513b7946b&language=en-US&query=' + value + '&page=1&include_adult=false')
                        .then((r) => {
                            this.results = r.data;
                            this.movieId = this.results.results[0].id;
                            this.fetch();
                        }).catch((error) => {
                            console.error(error);
                        }).finally(() => {
                            this.loading = false;
                        }),
                    500);
            }
        },
        fetch() {
            axios.all([
                axios.get('https://api.themoviedb.org/3/movie/' + this.movieId + '/watch/providers?api_key=f904a8044e5207e6d815f07513b7946b'),
                axios.get('https://api.themoviedb.org/3/movie/' + this.movieId + '?api_key=f904a8044e5207e6d815f07513b7946b&language=fr-FR'),
            ]).then(axios.spread((
                providers,
                movie
            ) => {
                this.providers = providers.data;
                this.movie = movie.data;
                console.log(this.movie);
            })).catch((error) => {
                console.error(error);
            }).finally(() => {
                this.loading = false;
            });
        },
    }
}
</script>

<style lang="scss">
.white--text {
    color: white !important;
}
</style>