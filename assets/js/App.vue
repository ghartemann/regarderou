<template>
    <v-container class="d-flex flex-column justify-center test">
        <v-row class="d-flex flex-column justify-center">
            <v-col class="d-flex">
                <h1 class="align-self-end white--text">Regarderou</h1>
            </v-col>
            <v-col>
                <v-autocomplete
                    @update:search="search($event)"
                    v-model="input"
                    :items="results.results"
                    :loading="loading"
                    :placeholder="this.placeholder"
                    clearable
                    hide-selected
                    autofocus
                    color="white"
                    hide-no-data
                    variant="outlined"
                >
                </v-autocomplete>
            </v-col>
        </v-row>

        <v-row v-if="providers && results.results && input !== null" justify="space-around">
            <v-card  width="1000">
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
                        <h2>Disponible sur :</h2>
                        <div class="d-flex">
                            <img :src="'https://www.themoviedb.org/t/p/original/' + provider.logo_path" :alt="provider.provider_name" class="rounded-lg mr-2" width="64">
                            <div>{{ provider.provider_name }}</div>
                        </div>
                    </div>
                    <div v-if="providers && providers.results && providers.results.FR && !providers.results.FR.flatrate">Ce film n'est pas disponible en streaming.</div>
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
            searchTimeout: null,
            placeholder: "Dune"
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
            } else {
                this.loading = false;
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

.test {
    min-height: 100vh;
}
</style>
