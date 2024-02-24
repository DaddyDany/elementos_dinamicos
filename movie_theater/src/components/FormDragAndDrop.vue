<template>
  <div
    class="container drag-item"
    draggable="true"
    @dragstart="handleDragStart"
    @dragend="handleDragEnd"
    id="formularioDrag"
  >
    <form id="form">
      <div v-if="errors.length">
        <b-alert show variant="danger">
          <b>Por favor, corrija el(los) siguiente(s) error(es):</b>
          <ul>
            <li v-for="error in errors" :key="error">{{ error }}</li>
          </ul>
        </b-alert>
      </div>
      <div class="row">
        <div class="col">
          <div class="fieldContainer">
            <div class="labelContainer">
              <label for="name">Nombre: </label>
            </div>
            <div class="inputContainer">
              <input id="name" v-model="name" type="text" name="name" />
            </div>
          </div>
        </div>
        <div class="col">
          <div class="fieldContainer">
            <div class="labelContainer">
              <label for="directorName">Nombre del Director: </label>
            </div>
            <div class="inputContainer">
              <input
                id="directorName"
                v-model="directorName"
                name="directorName"
                type="text"
              />
            </div>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col">
          <div class="fieldContainer">
            <div class="labelContainer">
              <label for="publicationDate">Fecha de publicación: </label>
            </div>
            <div class="inputContainer">
              <input
                id="publicationDate"
                v-model="publicationDate"
                name="publicationDate"
                type="date"
              />
            </div>
          </div>
        </div>
        <div class="col">
          <div class="fieldContainer">
            <div class="labelContainer">
              <label for="category">Categoría</label>
            </div>
            <div class="inputContainer">
              <select id="category" v-model="selected_category" name="category">
                <option
                  v-for="category in categories"
                  :key="category.id"
                  :value="category.id"
                >
                  {{ category.name }}
                </option>
              </select>
            </div>
          </div>
        </div>
      </div>
      <div class="fieldContainer">
        <div class="labelContainer">
          <label for="description">Descripción: </label>
        </div>
        <div class="inputContainer">
          <textarea
            id="description"
            v-model="description"
            name="description"
            rows="3"
          ></textarea>
        </div>
      </div>
    </form>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      errors: [],
      name: null,
      description: null,
      selected_category: null,
      directorName: null,
      publicationDate: null,
      categories: [],
    };
  },
  methods: {
    checkForm() {
      if (
        this.name &&
        this.description &&
        this.selected_category &&
        this.directorName &&
        this.publicationDate
      ) {
        return true;
      }
      this.errors = [];
      if (!this.name) {
        this.errors.push("El nombre es obligatorio.");
      }
      if (!this.description) {
        this.errors.push("La descripcion es obligatoria.");
      }
      if (!this.selected_category) {
        this.errors.push("Debes seleccionar una categoria.");
      }
      if (!this.directorName) {
        this.errors.push("El nombre del director es obligatorio");
      }
      if (!this.publicationDate) {
        this.errors.push("La fecha de publicación es obligatoria");
      }
    },
    clearErrors() {
      this.errors = [];
    },
    handleDragStart(event) {
      event.dataTransfer.setData(
        "application/json",
        JSON.stringify({
          name: this.name,
          description: this.description,
          directorName: this.directorName,
          publicationDate: this.publicationDate,
          category: {
            id: this.selected_category,
          },
        })
      );
    },
    handleDragEnd(event) {
      if (this.checkForm()) {
        this.clearErrors();
        this.name = null;
        this.description = null;
        this.selected_category = null;
        this.directorName = null;
        this.publicationDate = null;
      } else {
        console.log("Aún faltan datos por establecer");
      }
    },
  },
  mounted() {
    axios
      .get("http://localhost:8080/api/categories")
      .then((response) => {
        this.categories = response.data;
      })
      .catch((e) => {
        console.error("Error en la peticion: ", e);
      });
  },
};
</script>

<style>
#formularioDrag {
  background-color: #e6e6e6;
  border-radius: 10px;
  margin-top: 50px;
}

.buttonsContainer {
  width: 100%;
  display: flex;
  justify-content: space-evenly;
}

.buttonsContainer button {
  width: 30%;
  padding: 10px 20px;
  border-radius: 10px;
  border: none;
  margin-bottom: 10px;
  margin-top: 10px;
  transition: width 0.3s;
}

.buttonsContainer button:hover {
  width: 35%;
}

#botonEnviar {
  background-color: rgb(51, 139, 240);
  color: white;
}

#form {
  width: 100%;
  padding: 10px;
}

.fieldContainer {
  width: 100%;
  margin-bottom: 20px;
}

.labelContainer {
  margin-bottom: 10px;
}

.inputContainer {
  width: 100%;
}

.inputContainer input {
  padding: 10px;
  width: 100%;
  border: 2px solid #ccc;
  border-radius: 10px;
  background-color: #f9f9f9;
  color: #333;
  outline: none;
}

.inputContainer input:focus {
  border-color: #2b2b2b;
}
.inputContainer textarea {
  padding: 10px;
  width: 100%;
  border: 2px solid #ccc;
  border-radius: 10px;
  background-color: #f9f9f9;
  color: #333;
  outline: none;
}

.inputContainer textarea:focus {
  border-color: #2b2b2b;
}

.inputContainer select {
  padding: 10px;
  width: 100%;
  border: 2px solid #ccc;
  border-radius: 10px;
  background-color: #f9f9f9;
  color: #333;
  outline: none;
}

.inputContainer select:focus {
  border-color: #2b2b2b;
}
</style>
