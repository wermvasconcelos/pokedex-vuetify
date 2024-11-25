<template>
  <v-app>
    <v-container>
      <h1>Pokédex - Vuetify</h1>
      <br />
      <Loading v-if="initialLoading" />
      <!-- Campo de busca -->
      <v-text-field 
        v-if="!initialLoading" 
        v-model="searchPokemon" 
        label="Buscar Pokémon" 
        class="mb-4"
      ></v-text-field>
      
      <v-row>
        <!-- Itera apenas sobre os Pokémon filtrados -->
        <v-col 
          v-for="pokemon in filteredPokemons" 
          :key="pokemon.name" 
          cols="12" 
          sm="6" 
          md="3"
        >
          <v-card @click="openDialog(pokemon)">
            <v-card-title class="d-flex justify-space-between align-center">
              <span>{{ pokemon.name }}</span>
              <div>
                <v-chip 
                  v-for="(type, index) in pokemon.types" 
                  :key="index" 
                  :color="getTypeClass(type)" 
                  class="ma-1"
                  outlined
                >
                  {{ type }}
                </v-chip>
              </div>
            </v-card-title>
            <v-card-subtitle>Nº {{ pokemon.id }}</v-card-subtitle>
            <v-img 
              :src="pokemon.sprites.front_default" 
              alt="{{ pokemon.name }}" 
              height="150" 
            />
          </v-card>
        </v-col>
      </v-row>

      <!-- Botão para mostrar mais -->
      <v-container class="d-flex justify-center align-center">
        <v-btn 
          v-if="!initialLoading && pokemons.length > visiblePokemons.length" 
          @click="showMorePokemons"
          color="primary" 
          class="mt-4"
        >
          Mostrar mais
        </v-btn>
      </v-container>

      <!-- Modal de Detalhes do Pokémon -->
      <v-dialog v-model="dialog" max-width="500px">
        <v-card>
          <v-card-title>
            <span class="headline">{{ selectedPokemon?.name }}</span>
          </v-card-title>
          <v-card-subtitle>
            Nº {{ selectedPokemon?.id }}
          </v-card-subtitle>
          <v-card-text>
            <v-img 
              :src="selectedPokemon?.sprites_gif.front_default" 
              alt="Imagem do Pokémon" 
              height="200"
            ></v-img>
          </v-card-text>
          <v-card-actions>
            <v-btn color="primary" @click="dialog = false">Fechar</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-container>
  </v-app>
</template>

<script lang="ts" setup>
import { computed, ref } from 'vue';
import axios from 'axios';

import Loading from '@/components/Loading.vue';

interface Pokemon {
  name: string;
  id: number;
  sprites: {
    front_default: string;
  };
  sprites_gif: {
    front_default: string;
  };
  types: string[];
}

const searchPokemon = ref('');
const pokemons = ref<Pokemon[]>([]);
const visiblePokemons = ref<Pokemon[]>([]);
const initialLoading = ref(true);
const dialog = ref(false);
const selectedPokemon = ref<Pokemon | null>(null);

const fetchPokemons = async () => {
  initialLoading.value = true;

  try {
    const requests = Array.from({ length: 1025 }, (_, i) =>
      axios.get(`https://pokeapi.co/api/v2/pokemon/${i + 1}`)
    );

    const responses = await Promise.all(requests);

    pokemons.value = responses.map((response) => ({
      name: response.data.name,
      id: response.data.id,
      sprites: response.data.sprites,
      sprites_gif: response.data.sprites.other.showdown,
      types: response.data.types.map((type: { type: { name: string } }) => type.type.name),
    }));

    visiblePokemons.value = pokemons.value.slice(0, 20);
  } catch (error) {
    console.error('Erro ao buscar os Pokémon:', error);
  }

  initialLoading.value = false;
};

const showMorePokemons = () => {
  const currentLength = visiblePokemons.value.length;
  visiblePokemons.value = pokemons.value.slice(0, currentLength + 10);
};

const openDialog = (pokemon: Pokemon) => {
  selectedPokemon.value = pokemon;
  dialog.value = true;
};

const getTypeClass = (type: string) => {
  const typeColors: { [key: string]: string } = {
    grass: 'green-accent-4',
    poison: 'deep-purple-accent-1',
    fire: 'red-accent-4',
    water: 'light-blue-accent-4',
    electric: 'yellow-accent-2',
    bug: 'lime-darken-1',
    flying: 'white',
    fighting: 'deep-orange-lighten-1',
    ground: 'brown-lighten-2',
    rock: 'blue-grey-lighten-5',
    psychic: 'deep-purple-lighten-3',
    ice: 'light-blue-lighten-5',
    ghost: 'blue-grey-lighten-2',
    steel: 'blue-grey-darken-4',
    dragon: 'teal-accent-4',
    dark: 'transparent',
    fairy: 'pink-accent-3',
  };

  return typeColors[type] || 'grey lighten-2';
};

// Computed para retornar a lista filtrada de Pokémon
const filteredPokemons = computed(() => {
  if (!searchPokemon.value) return visiblePokemons.value;

  return pokemons.value.filter((pokemon) =>
    pokemon.name.toLowerCase().includes(searchPokemon.value.toLowerCase())
  );
});

// Chama a função ao montar o componente
fetchPokemons();
</script>
