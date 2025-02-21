<script lang="ts" setup>
export interface Character {
  info: Info;
  results: Result[];
}

export interface Info {
  count: number;
  pages: number;
  next: string;
  prev: null;
}

export interface Result {
  id: number;
  name: string;
  status: string;
  species: string;
  type: string;
  gender: string;
  origin: Location;
  location: Location;
  image: string;
  episode: string[];
  url: string;
  created?: Date;
  show: Boolean;
}
export interface Location {
  name: string;
  url: string;
}
// const characters = localStorage.setItem("myCat", "Tom");
const { data, refresh } = await useFetch<Character>('https://rickandmortyapi.com/api/character')



const characterEdit = ref<Result>({
  id: 0,
  name: '',
  status: '',
  species: '',
  type: '',
  gender: '',
  origin: {
    name: '',
    url: '',
  },
  show: false,
  location: {
    name: '',
    url: '',
  },
  image: '',
  episode: [],
  url: '',
  created: new Date(),
})

const characterTemplate = ref<Result>({
  id: 0,
  name: '',
  status: '',
  species: '',
  type: '',
  gender: '',
  origin: {
    name: '',
    url: '',
  },
  show: false,
  location: {
    name: '',
    url: '',
  },
  image: '',
  episode: [],
  url: '',
  created: new Date(),
})

const nameModal = ref("New character")
const dialog = ref(false)
const indexCharacterList = ref<number | null>(null)
const onClickNewCharacter = () => {
  nameModal.value = "New Character"
  dialog.value = true
  characterEdit.value = characterTemplate.value
  indexCharacterList.value = null
}

const onClickEditCharacter = (item: Result, index: number) => {
  nameModal.value = "Edit Character"
  dialog.value = true
  characterEdit.value = { ...item }
  indexCharacterList.value = index
}

const saveCharacter = () => {
  if (!data.value) throw new Error("No data")
  if (!data.value.results) throw new Error("No data")
  if (indexCharacterList.value === null) {
    data.value.results.push(characterEdit.value)
  } else {
    data.value.results[indexCharacterList.value] = { ...characterEdit.value }
  }
  dialog.value = false
  localStorage.setItem("characters", JSON.stringify(data.value))
}

const onClickDeleteCharacter = (index: number) => {
  if (!data.value) throw new Error("No data")
  if (!data.value.results) throw new Error("No data")
  data.value.results.splice(index, 1)
  localStorage.setItem("characters", JSON.stringify(data.value))

}

onMounted(() => {
  const characters = localStorage.getItem("characters");
  if (characters) {
    data.value = JSON.parse(characters)
  } else {
    refresh()
    if (data.value && data.value.results) {
      data.value.results.map((item) => {
        item.show = false
      })
    }
  }

})

</script>
<template>
  <v-container fluid>
    <div>
      <v-row>
        <v-col cols="12" class="d-flex justify-center">
          <v-btn size="x-large" block color="blue" @click="onClickNewCharacter">Nuevo</v-btn>
        </v-col>

        <v-col v-for="(item, index) in data?.results" :key="item.id">
          <v-card class="mx-auto" max-width="344" min-width="244">
            <v-img height="200px" :src="item.image" cover></v-img>
            <v-card-title>
              {{ item.name }}
            </v-card-title>
            <v-card-subtitle>
              {{ item.species }}
            </v-card-subtitle>
            <v-card-actions>
              <v-btn color="blue-lighten-2" text="Editar" variant="outlined"
                @click="onClickEditCharacter(item, index)"></v-btn>
              <v-btn color="red-lighten-2" variant="outlined" icon="mdi-delete"
                @click="onClickDeleteCharacter(index)"></v-btn>
              <v-spacer></v-spacer>
              <v-btn :icon="item.show ? 'mdi-chevron-up' : 'mdi-chevron-down'" @click="item.show = !item.show"></v-btn>

            </v-card-actions>
            <v-expand-transition>
              <div v-show="item.show">
                <v-divider></v-divider>
                <v-card-text>
                  <b>Genero:</b> {{ item.gender }}
                  <br />
                  <b>Especie:</b> {{ item.species }}
                  <br />
                  <b>Estado:</b> {{ item.status }}
                  <br />
                  <b>Origen:</b> {{ item.origin.name }}
                </v-card-text>
              </div>
            </v-expand-transition>
          </v-card>
        </v-col>
      </v-row>

    </div>
    <v-dialog v-model="dialog" max-width="500">
      <v-card>
        <v-card-title>{{ nameModal }}{{ indexCharacterList }}</v-card-title>
        <v-card-text>
          <v-text-field label="Nombre del personaje" v-model="characterEdit.name"></v-text-field>
          <v-text-field label="Especie" v-model="characterEdit.species"></v-text-field>
          <v-text-field label="Estado" v-model="characterEdit.status"></v-text-field>
          <v-text-field label="Origen" v-model="characterEdit.origin.name"></v-text-field>
          <v-text-field label="Tipo" v-model="characterEdit.type"></v-text-field>
          <v-text-field label="Género" v-model="characterEdit.gender"></v-text-field>
          <v-text-field label="Imagen" v-model="characterEdit.image"></v-text-field>
          <v-text-field label="URL" v-model="characterEdit.url"></v-text-field>


          <small class="text-grey">* This doesn't actually save.</small>
        </v-card-text>

        <v-card-actions>
          <v-btn color="red" variant="text" @click="dialog = false">Cancelar</v-btn>
          <v-spacer></v-spacer>

          <v-btn color="primary" variant="elevated" @click="saveCharacter()">
            Guardar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

  </v-container>

</template>
