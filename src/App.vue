<script setup>
import { ref, onMounted } from 'vue';
import Movies from './components/Movies.vue';

const movies = ref({});
const loading = ref(false);
const added = ref(false);
const deleted = ref(false);
const error = ref(false);
const delError = ref(false);
const addError = ref(false);
const info = ref('');
const btnAddMovieClicked = ref(false);
const title = ref('');
const releaseDate = ref('');
const director = ref('');

const url =
  'https://vue-movies-app-aa461-default-rtdb.europe-west1.firebasedatabase.app/movie.json';

onMounted(() => {
  handleLoadMovies();
});

async function handleLoadMovies() {
  added.value = false;
  deleted.value = false;
  loading.value = true;
  delError.value = false;
  addError.value = false;

  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error('Something went wrong!');
    }
    const data = await response.json();
    movies.value = data;
    loading.value = false;
  } catch (err) {
    loading.value = false;
    error.value = err;
  }
}

function handleSubmit(event) {
  event.preventDefault();
}

async function handleAddMovie() {
  btnAddMovieClicked.value = true;
  deleted.value = false;
  error.value = false;
  delError.value = false;

  if (title.value.trim().length === 0) {
    return false;
  }

  let movie = {
    title: title.value,
    release_date: releaseDate.value,
    director: director.value,
  };

  const postMetod = {
    method: 'POST',
    body: JSON.stringify(movie),
    headers: {
      'Content-Type': 'application/json',
    },
  };

  try {
    const response = await fetch(url, postMetod);
    if (!response.ok) {
      throw new Error('Something went wrong!');
    }
    const data = await response.json();
    handleLoadMovies();
    added.value = true;
    info.value = movie.title;

    title.value = '';
    releaseDate.value = '';
    director.value = '';
    btnAddMovieClicked.value = false;
  } catch (err) {
    loading.value = false;
    addError.value = err;
  }
}

async function deleteAllMoviesHandler() {
  added.value = false;
  error.value = false;
  addError.value = false;

  const delMethod = {
    method: 'DELETE',
    headers: {
      'Content-Type': 'application/json',
    },
  };

  try {
    const response = await fetch(url, delMethod);
    if (!response.ok) {
      throw new Error('Something went wrong!');
    }
    const data = await response.json();
    handleLoadMovies();
    deleted.value = true;
  } catch (err) {
    loading.value = false;
    delError.value = err;
  }
}

const focusEventHandler = function () {
  btnAddMovieClicked.value = false;
};
</script>

<template>
  <div class="container">
    <h1 class="my-4 text-center">Vue Movie Catalogue App</h1>
    <p>
      Raw JSON (hosted in a Firebase Realtime Database):<br />
      <a
        href="https://vue-movies-app-aa461-default-rtdb.europe-west1.firebasedatabase.app/movie.json"
        target="_blank"
        >https://vue-movies-app-aa461-default-rtdb.europe-west1.firebasedatabase.app/movie.json</a
      >
    </p>
    <form @submit="handleSubmit">
      <fieldset>
        <legend>Enter new movie data:</legend>
        <input
          class="form-control mt-3"
          :class="{ empty: title.trim().length === 0 && btnAddMovieClicked }"
          type="text"
          id="title"
          placeholder="Enter the movie title..."
          v-model="title"
          @focus="() => focusEventHandler()"
        />
        <input
          class="form-control mt-3"
          type="date"
          id="release_date"
          v-model="releaseDate"
        />
        <input
          class="form-control my-3"
          type="text"
          id="director"
          placeholder="Enter the name of the director..."
          v-model="director"
        />
        <button class="btn btn-success my-3" @click="handleAddMovie">
          Add Movie
        </button>
        <p v-if="addError" class="d-inline-block text-danger">
          &nbsp; {{ addError }}
        </p>
        <p v-else-if="added" class="d-inline text-secondary">
          &nbsp; Successfully added movie:
          {{ info || '[Movie title not entered!]' }}
        </p>
        <br />

        <button class="btn btn-danger my-3" @click="deleteAllMoviesHandler">
          Delete All Movies
        </button>
        <p v-if="delError" class="d-inline-block text-danger">
          &nbsp; {{ delError }}
        </p>
        <p v-else-if="deleted" class="d-inline text-secondary">
          &nbsp; All movies successfully deleted!
        </p>
        <br />

        <p class="text-secondary loading" :class="{ visible: loading }">
          Movies loading...
        </p>
        <p v-if="error" class="text-danger">{{ error }}</p>
      </fieldset>
    </form>

    <Movies
      v-if="movies"
      :element="movies"
      @onDeleteMovie="handleLoadMovies"
    ></Movies>
    <p v-else class="text-danger">No movies found.</p>

    <footer class="footer">
      <p>Coded by it-drafter</p>
      <p>December 2022.</p>
    </footer>
  </div>
</template>

<style scoped>
@import 'https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/css/bootstrap.min.css';

.loading {
  visibility: hidden;
}
.loading.visible {
  visibility: visible;
}
.footer {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 30px 0;
}
.footer p {
  margin: 0;
  font-weight: bold;
}
.form-control.empty {
  border-color: red;
  background: rgb(255, 210, 210);
}
</style>
