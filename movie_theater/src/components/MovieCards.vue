<template>
  <div
    class="container"
    style="height: 100vh"
    @drop="handleDrop"
    @dragover.prevent
  >
    <FormDragAndDrop
      v-show="showElement"
      ref="formDragAndDrop"
    ></FormDragAndDrop>
    <div style="display: flex; margin-top: 100px; padding-bottom: 20px">
      <h1 style="margin-bottom: 0">Peliculas</h1>
      <Modal @registroExitoso="actualizarPeliculas"></Modal>
    </div>
    <div style="margin-bottom: 50px; width: 50%">
      <div>
        <form @submit.prevent="buscarPeliculas">
          <input
            id="search"
            name="search"
            v-model="search"
            type="search"
            style="padding: 10px; border-radius: 8px; width: 80%"
            placeholder="Nombre pelicula, nombre director, categoria"
          />
          <button
            style="padding: 10px; border-radius: 8px; width: 20%"
            type="submit"
          >
            Buscar
          </button>
        </form>
      </div>
      <div style="margin-top: 20px">
        <p style="font-size: 1.5rem; margin-bottom: 10px">
          Fecha de publicación
        </p>
        <form @submit.prevent="buscarPeliculasPorFecha">
          <div style="margin-bottom: 10px">
            <label for="startDate" style="margin-right: 10px"
              >Buscar desde:</label
            >
            <input
              type="date"
              style="
                padding: 10px;
                border-radius: 8px;
                width: 30%;
                margin-right: 50px;
              "
              id="startDate"
              v-model="startDate"
            />
            <label for="endDate" style="margin-right: 10px">Hasta:</label>
            <input
              type="date"
              style="padding: 10px; border-radius: 8px; width: 30%"
              id="endDate"
              v-model="endDate"
            />
          </div>
          <div style="margin-top: 17px">
            <button
              type="submit"
              style="padding: 10px; border-radius: 8px; width: 40%"
              :disabled="!startDate || !endDate"
            >
              Buscar por rango de fechas
            </button>
          </div>
        </form>
      </div>
      <div>
        <button
          @click="ordenarPeliculasDesc"
          style="
            padding: 10px;
            border-radius: 8px;
            width: 50%;
            margin-top: 20px;
          "
        >
          Mostrar todas las peliculas de forma descendente por fecha de
          publicación
        </button>
      </div>
    </div>
    <div class="text-center" id="loaderSpinner" v-if="loading">
      <b-spinner class="spinner" type="grow"></b-spinner>
      <b-spinner class="spinner" type="grow"></b-spinner>
      <b-spinner class="spinner" type="grow"></b-spinner>
      <b-spinner class="spinner" type="grow"></b-spinner>
      <b-spinner class="spinner" type="grow"></b-spinner>
    </div>
    <div v-if="peliculas">
      <TransitionGroup name="zoomDown" tag="div" class="row">
        <div
          class="col-4 movieCards"
          v-for="pelicula in peliculas"
          :key="pelicula.id"
        >
          <div>
            <b-card
              :title="pelicula.name"
              img-src="https://picsum.photos/600/300/?image=25"
              img-alt="Image"
              img-top
              tag="article"
              style="max-width: 20rem"
              class="mb-2"
            >
              <b-card-text class="description">
                {{ pelicula.description }}
              </b-card-text>

              <div>
                <p class="CustomBadge">{{ pelicula.category.name }}</p>
                <p><b>Director: </b>{{ pelicula.directorName }}</p>
                <p>
                  <b>Fecha de publicación: </b
                  >{{ dateFormat(pelicula.publicationDate) }}
                </p>
              </div>
              <div style="width: 100%; display: flex">
                <div
                  style="
                    display: flex;
                    width: 100%;
                    justify-content: space-around;
                  "
                >
                  <b-button
                    variant="danger"
                    style="width: 45%; padding: 8px"
                    @click="deleteMovie(pelicula.id)"
                    >Eliminar</b-button
                  >
                  <EditModal
                    style="width: 45%"
                    :key="'modalEdicion_' + pelicula.id"
                    :pelicula="pelicula"
                    @actualizacionExitosa="actualizarPeliculas"
                  ></EditModal>
                </div>
              </div>
            </b-card>
          </div>
        </div>
      </TransitionGroup>
    </div>
    <div v-else>
      <h3 class="text-center">Sin resultados</h3>
    </div>
  </div>
</template>

<script>
import Modal from "./Modal.vue";
import EditModal from "./EditModal.vue";
import axios from "axios";
import FormDragAndDrop from "./FormDragAndDrop.vue";

export default {
  name: "MovieCards",
  components: {
    Modal,
    EditModal,
    FormDragAndDrop,
  },
  data() {
    return {
      peliculas: [],
      loading: null,
      search: null,
      endDate: null,
      startDate: null,
      showElement: true,
      lastScrollPosition: 0,
      loaded: false,
    };
  },
  methods: {
    actualizarPeliculas() {
      axios
        .get("http://localhost:8080/api/movies")
        .then((response) => {
          this.peliculas = response.data;
          this.loading = false;
        })
        .catch((e) => {
          console.error("Error en la peticion: ", e);
        });
    },
    handleDrop(event) {
      event.preventDefault();
      const formData = JSON.parse(
        event.dataTransfer.getData("application/json")
      );
      console.log(formData);
      if (
        formData.name &&
        formData.description &&
        formData.directorName &&
        formData.publicationDate &&
        formData.category.id
      ) {
        axios
          .post("http://localhost:8080/api/movies", formData)
          .then((response) => {
            this.$swal(
              "¡Éxito!",
              "La película se registró correctamente",
              "success"
            );
            this.actualizarPeliculas();
          })
          .catch((error) => {
            console.error("Error al registrar la película:", error);
            this.$swal(
              "Error",
              "Hubo un error al registrar la película",
              "error"
            );
          });
      }
    },
    buscarPeliculas() {
      this.loading = true;
      axios
        .get(`http://localhost:8080/api/movies/search?search=${this.search}`)
        .then((response) => {
          this.peliculas = response.data;
          this.loading = false;
        })
        .catch((error) => {
          console.error("Error en la búsqueda: ", error);
          this.loading = false;
        });
    },
    ordenarPeliculasDesc() {
      this.loading = true;
      axios
        .get("http://localhost:8080/api/movies/latest")
        .then((response) => {
          this.peliculas = response.data;
          this.loading = false;
        })
        .catch((e) => {
          console.error("Error en la peticion: ", e);
        });
    },
    deleteMovie(id) {
      this.$swal({
        title: "¿Estas seguro?",
        text: "No podras revertir este cambio",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        cancelButtonText: "cancelar",
        confirmButtonText: "Si, eliminar",
      }).then((result) => {
        if (result.isConfirmed) {
          axios
            .delete(`http://localhost:8080/api/movies/${id}`)
            .then((response) => {
              this.$swal({
                title: "Eliminada",
                text: "La pelicula ha sido eliminada con exito",
                icon: "success",
              });
              this.actualizarPeliculas();
            })
            .catch((error) => {
              console.error(error);
            });
        }
      });
    },
    buscarPeliculasPorFecha() {
      axios
        .get(
          `http://localhost:8080/api/movies/searchDateRange?startDate=${this.startDate}&endDate=${this.endDate}`
        )
        .then((response) => {
          this.peliculas = response.data;
          console.log("Peliculas encontradas");
        })
        .catch((error) => {
          console.error("Error al buscar películas");
        });
    },
    dateFormat(date) {
      let dateParts = date.split("-");
      let year = dateParts[0];
      let month = dateParts[1];
      let day = dateParts[2];
      let newDate = day + "-" + month + "-" + year;
      return newDate;
    },
    onScroll() {
      const currentScrollPosition =
        window.pageYOffset || document.documentElement.scrollTop;
      console.log(currentScrollPosition);
      if (Math.abs(currentScrollPosition - this.lastScrollPosition) < 50) {
        return;
      }
      this.showElement = currentScrollPosition < this.lastScrollPosition;
      this.lastScrollPosition = currentScrollPosition;
    },
  },
  mounted() {
    this.loading = true;
    window.addEventListener("scroll", this.onScroll);
    setTimeout(() => {
      axios
        .get("http://localhost:8080/api/movies")
        .then((response) => {
          this.peliculas = response.data;
        })
        .catch((e) => {
          console.error("Error en la peticion: ", e);
        })
        .finally(() => {
          this.loading = false;
        });
    }, 1000);
  },
  beforeDestroy() {
    window.removeEventListener("scroll", this.onScroll);
  },
};
</script>
<style>
.description {
  height: 50px;
}
.CustomBadge {
  background-color: black;
  border-radius: 50px;
  color: white;
  text-align: center;
  width: 50%;
}
.movieCards {
  display: flex;
  justify-content: center;
}
#loaderSpinner {
  height: 75%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.spinner {
  margin: 7px;
}

.hidden {
  display: none;
}
</style>
