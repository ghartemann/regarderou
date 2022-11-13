<template>
    <v-container>
        <h1>Regarderou</h1>

        <v-row>
            <v-col class="d-flex">
                <v-text-field
                    label="Movie"
                    variant="outlined"
                    v-model="movieId"
                ></v-text-field>
                <v-btn
                    icon="mdi-magnify"
                    color="primary"
                    @click="fetch"
                ></v-btn>
            </v-col>
        </v-row>

        <v-row>
            <v-col>
                <div>
                    <div v-if="providers && providers.results && providers.results.FR" v-for="provider in providers.results.FR.flatrate">
                        <h2>{{provider.provider_name}}</h2>
                        <img :src="'https://www.themoviedb.org/t/p/original/' + provider.logo_path">
                        <div>{{movie.original_title}}</div>
                    </div>
                    <div v-if="providers && providers.results && providers.results.FR && !providers.results.FR.flatrate">Pas de résultats.</div>
                </div>
            </v-col>
        </v-row>
    </v-container>
</template>


<script>
import axios from "axios";

export default {
    name: 'App',
    data() {
        return {
            loading: true,
            providers: {},
            movie: {},
            movieId: null,
        }
    },
    methods: {
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
        }
    }
}
</script>

