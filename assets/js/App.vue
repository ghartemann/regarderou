<template>
    <v-container
        fluid class="d-flex flex-column justify-center test"
        :style="providers && results.results && input && input !== '' ? 'background-image:' + background + 'background-size: cover; width: 100vw;' : ''">

        <transition-group name="list" mode="out-in">
            <v-row key="1">
                <v-spacer></v-spacer>
                <v-col class="d-flex flex-column justify-center col-6">
                    <div>
                        <v-autocomplete
                            @update:search="search($event)"
                            v-model="input"
                            :items="results.results"
                            :loading="loading"
                            :placeholder="placeholder"
                            clearable
                            hide-selected
                            autofocus
                            color="white"
                            hide-no-data
                            return-object
                            rounded
                            variant="solo"
                            :menu-props="{ maxHeight: 200 }"
                        >
                        </v-autocomplete>
                    </div>
                </v-col>
                <v-spacer></v-spacer>
            </v-row>
    
            <v-row key="2" v-if="providers && results.results && input !== null" justify="space-around">
                <v-spacer></v-spacer>
                <v-col class="col-6">
                    <v-card>
                        <v-card-text>
                            <div v-if="providers && providers.results && providers.results.FR" v-for="provider in providers.results.FR.flatrate">
                                <div class="d-flex">
                                    <img :src="'https://www.themoviedb.org/t/p/original/' + provider.logo_path" :alt="provider.provider_name" class="rounded-lg mr-2" width="64">
                                </div>
                            </div>
                            <div v-if="providers && providers.results && providers.results.FR && !providers.results.FR.flatrate">Ce film n'est pas disponible en streaming.</div>
                        </v-card-text>
                    </v-card>
                </v-col>
                <v-spacer></v-spacer>
            </v-row>
        </transition-group>
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
            trending: [],
            placeholder: '',
            placeholderIndex: null,
            loadingTrending: true,
            background: "none"
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
                    () => axios.get('https://api.themoviedb.org/3/search/movie?api_key=' + process.env.API_KEY + '&language=en-US&query=' + value + '&page=1&include_adult=false')
                        .then((r) => {
                            this.results = r.data;
                            this.movieId = this.results.results[0].id;

                            this.background = "url('https://www.themoviedb.org/t/p/original" + this.results.results[0].backdrop_path + "');";
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
                axios.get('https://api.themoviedb.org/3/movie/' + this.movieId + '/watch/providers?api_key=' + process.env.API_KEY),
                axios.get('https://api.themoviedb.org/3/movie/' + this.movieId + '?api_key=' + process.env.API_KEY + '&language=fr-FR'),
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
        getTrending() {
            this.loadingTrending = true;

            axios.get('https://api.themoviedb.org/3/trending/movie/week?api_key=' + process.env.API_KEY)
                .then((r) => {
                    for (let i = 0; i < r.data.results.length; i++) {
                        this.trending.push(r.data.results[i].title);
                    }
                    this.getRandomPlaceholder();
                    this.autoChangePlaceholder();
                }).catch((error) => {
                    console.error(error);
            }).finally(() => {
                this.loadingTrending = false;
            });
        },
        getRandomPlaceholder() {
            this.placeholder = this.trending[Math.floor(Math.random() * (this.trending.length))];
            this.placeholderIndex = this.trending.findIndex((x) => x === this.placeholder);
        },
        autoChangePlaceholder() {
            setInterval(this.changePlaceholder, 1500);
        },
        changePlaceholder() {
            this.placeholderIndex++;

            if (this.placeholderIndex === this.trending.length) {
                this.placeholderIndex = 0;
            }

            this.placeholder = this.trending[this.placeholderIndex];
        }
    },
    created() {
        this.getTrending();
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

.list-move, /* apply transition to moving elements */
.list-enter-active,
.list-leave-active {
    transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
    opacity: 0;
    transform: translateX(30px);
}

/* ensure leaving items are taken out of layout flow so that moving
   animations can be calculated correctly. */
.list-leave-active {
    position: absolute;
}
</style>
