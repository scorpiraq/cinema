<template>
  <h1
    class="flex flex-row text-3xl font-bold text-center py-4 bg-red-700 mb-5 justify-between"
  >
    <div class="text-left w-1/2">
      <a @click="goBack" class="text-white font-bold my-2 cursor-pointer">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          fill="none"
          viewBox="0 0 24 24"
          stroke-width="2"
          stroke="currentColor"
          class="w-10 h-6"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            d="M9 15L3 9m0 0l6-6M3 9h12a6 6 0 010 12h-3"
          />
        </svg>
      </a>
    </div>
    <div class="text-left w-1/2">Cinema Movies</div>
  </h1>
  <div>
    <div class="container mx-auto px-4 flex items-center justify-between">
      <!-- <h1 class="text-white text-2xl font-bold">Movie Details</h1> -->
    </div>

    <!-- Movie Details -->
    <div class="container mx-auto px-4 py-8 space-y-4">
      <!-- Movie Content -->
      <div v-if="movie">
        <div class="flex flex-col md:flex-row pb-4">
          <div class="md:w-1/3">
            <img
              v-if="movie.poster_path"
              :src="fullImageUrl(movie.poster_path)"
              :alt="movie.title"
              class="w-full mb-6 md:mb-0"
            />
            <div
              v-else
              class="w-full mb-6 md:mb-0 bg-gray-300"
              style="height: 500px"
            ></div>
          </div>
          <div class="md:w-2/3 md:pl-8">
            <h2 class="text-4xl font-bold mb-4">{{ movie.title }}</h2>
            <div class="flex items-center mb-4">
              <span class="mr-2">Rating:</span>
              <span class="text-yellow-500 flex justify-between">
                {{ movie.vote_average.toFixed(1) }}/10
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  fill="none"
                  viewBox="0 0 24 24"
                  stroke-width="1.5"
                  stroke="currentColor"
                  class="w-6 h-6"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    d="M11.48 3.499a.562.562 0 011.04 0l2.125 5.111a.563.563 0 00.475.345l5.518.442c.499.04.701.663.321.988l-4.204 3.602a.563.563 0 00-.182.557l1.285 5.385a.562.562 0 01-.84.61l-4.725-2.885a.563.563 0 00-.586 0L6.982 20.54a.562.562 0 01-.84-.61l1.285-5.386a.562.562 0 00-.182-.557l-4.204-3.602a.563.563 0 01.321-.988l5.518-.442a.563.563 0 00.475-.345L11.48 3.5z"
                  />
                </svg>
              </span>
            </div>
            <p class="text-lg mb-4">{{ movie.overview }}</p>
            <div class="flex items-center">
              <span class="mr-2">Release Date:</span>
              <span>{{ formatDate(movie.release_date) }}</span>
            </div>
            <div class="flex items-center mt-4">
              <span class="mr-2">Genre:</span>
              <span
                v-for="genre in movie.genres"
                :key="genre.id"
                class="px-2"
                >{{ genre.name }}</span
              >
            </div>
            <div class="flex items-center mt-4">
              <span class="mr-2">Duration:</span>
              <span>{{ formatDuration(movie.runtime) }}</span>
            </div>

            <div class="mt-8">
              <h3 class="text-2xl font-bold mb-4">Additional Information</h3>
              <p>
                <span class="font-bold">Original Language:</span>
                {{ capitalizedLanguage(movie.original_language) }}
              </p>
            </div>
          </div>
        </div>
        <!-- Movie Section -->
        <div class="grid grid-cols-4 gap-4">
          <div v-for="(video, index) in videos">
            <span class="font-bold text-white">
              Trailer {{ index + 1 }}

              <router-link
                :to="{ name: 'Player', params: { videoKey: video.key } }"
              >
                <img
                  :src="`https://img.youtube.com/vi/${video.key}/0.jpg`"
                  alt=""
                />
              </router-link>
            </span>
          </div>
        </div>
        <div class="bg-gray-900 py-16">
          <div class="container mx-auto px-4">
            <h2 class="text-3xl font-bold mb-8 text-center text-red">
              Related Movies
            </h2>
            <div
              class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-8"
            >
              <!-- Movie Card -->
              <div v-for="movie in movies">
                <router-link
                  :to="{ name: 'Details', params: { id: movie.id } }"
                >
                  <div
                    class="bg-gray-100 p-4 rounded shadow movie-card overflow-hidden transition hover:scale-105"
                  >
                    <img
                      :src="fullImageUrl(movie.backdrop_path)"
                      alt="Movie Poster"
                      class="mb-4 rounded transition hover:scale-150 sm:w-full"
                    />
                    <h3 class="text-lg font-bold text-red-400">
                      {{ movie.title }}
                    </h3>
                    <div class="mt-4">
                      <p class="text-gray-700">
                        <span class="font-bold">
                          Vote: {{ movie.vote_average.toFixed(1) * 10 }}% </span
                        >-
                        <span class="font-bold">
                          {{ formatDate(movie.release_date) }}
                        </span>
                      </p>
                    </div>
                    <router-link
                      :to="{ name: 'Details', params: { id: movie.id } }"
                      class="mt-4 inline-block bg-red-600 hover:bg-red-700 text-white font-bold py-2 px-4 rounded"
                    >
                      Details
                    </router-link>
                  </div>
                </router-link>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from "vue";
import axios from "@/http";
import { fullImageUrl } from "@/helpers";

const movie = ref(null);
const movies = ref([]);
const props = defineProps(["id"]);
const movieId = ref(props.id);
const videos = ref([]);

const getMovieDetails = async () => {
  const { data } = await axios.get(`/movie/${movieId.value}`);
  movie.value = data;

  const response = await axios.get("/discover/movie", {
    params: {
      with_genres: movie.value.genres[0].id,
    },
  });
  movies.value = response.data.results.slice(0, 8);
};

const formatDuration = (minutes) => {
  const hours = Math.floor(minutes / 60);
  const mins = minutes % 60;
  return `${hours}h ${mins}m`;
};

const formatDate = (date) => {
  const options = { year: "numeric", month: "long", day: "numeric" };
  return new Date(date).toLocaleDateString(undefined, options);
};

const capitalizedLanguage = (original_language) => {
  if (original_language) {
    return original_language.toUpperCase();
  }
  return "";
};

const fetchMovieVideos = async () => {
  const { data } = await axios.get(`/movie/${props.id}/videos`);
  videos.value = data.results.slice(0, 8);
};

const goBack = () => {
  router.back();
};

onMounted(() => {
  getMovieDetails();
  fetchMovieVideos();
});

watch(
  () => props.id,
  () => {
    movieId.value = props.id;
    getMovieDetails();
  }
);
</script>
