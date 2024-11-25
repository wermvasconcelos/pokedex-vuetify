<template>
  <v-app>
    <v-container>
      <h1>Pokédex - Vuetify</h1>
      <br>
      <Loading v-if="initialLoading"/>
      <v-row>
        <!-- Itera sobre a lista de Pokémon e exibe cada um -->
        <v-col v-for="pokemon in pokemons" :key="pokemon.name" cols="12" sm="6" md="3">
          <v-card @click="openDialog(pokemon)">
            <v-card-title class="d-flex justify-space-between align-center">
              <span>{{ pokemon.name }}</span>
              <div>
                <v-chip v-for="(type, index) in pokemon.types" :key="index" :color="getTypeClass(type)" class="ma-1"
                  outlined>
                  {{ type }}
                </v-chip>
              </div>
            </v-card-title>
            <v-card-subtitle>Nº {{ pokemon.id }}</v-card-subtitle>
            <v-img :src="pokemon.sprites.front_default" alt="{{ pokemon.name }}" height="150" />
          </v-card>
        </v-col>
      </v-row>

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
            <v-img :src="selectedPokemon?.sprites_gif.front_default" alt="Imagem do Pokémon" height="200"></v-img>
            <!-- Aqui você pode adicionar mais informações sobre o Pokémon -->
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
import { ref } from 'vue';
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
  types: string[]; // Array de strings com os tipos do Pokémon
}

// Array reativo para armazenar os Pokémon
const pokemons = ref<Pokemon[]>([]);

// Estado para configurar "loading"
const initialLoading = ref(true);

// Estado para controlar a exibição do modal
const dialog = ref(false);
const selectedPokemon = ref<Pokemon | null>(null);

// Função para buscar os primeiros 151 Pokémon
const fetchPokemons = async () => {
  initialLoading.value = true;

  try {
    // Cria um array de promessas para os IDs de 1 a 151
    const requests = Array.from({ length: 1025 }, (_, i) =>
      axios.get(`https://pokeapi.co/api/v2/pokemon/${i + 1}`)
    );

    // Aguarda a conclusão de todas as requisições
    const responses = await Promise.all(requests);

    // Extrai os dados relevantes de cada resposta
    pokemons.value = responses.map((response) => ({
      name: response.data.name,
      id: response.data.id,
      sprites: response.data.sprites,
      sprites_gif: response.data.sprites.other.showdown,
      types: response.data.types.map((type: { type: { name: string } }) => type.type.name), // Extraindo o nome de cada tipo
    }));

    console.log(pokemons.value);
  } catch (error) {
    console.error('Erro ao buscar os Pokémon:', error);
  }

  initialLoading.value = false;
};

// Função para abrir o modal com as informações do Pokémon selecionado
const openDialog = (pokemon: Pokemon) => {
  selectedPokemon.value = pokemon;
  dialog.value = true;
};

// Função para retornar a classe de cor do tipo
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

  return typeColors[type] || 'grey lighten-2'; // Padrão para tipos não encontrados
};

// Chama a função ao montar o componente
fetchPokemons();
</script>
