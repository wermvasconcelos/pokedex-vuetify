<template>
  <!-- <HelloWorld /> -->
  <v-app>
    <v-container>
      <h1>Pokédex - Vuetify</h1>
      <v-row>
        <v-col v-for="pokemon in pokemons" :key="pokemon.name" cols="12" md="4">
          <v-card>
            <v-card-title>{{ pokemon.name }}</v-card-title>
            <v-img :src="pokemon.sprites.front_default" alt="{{ pokemon.name }}" height="150" />
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script lang="ts" setup>
import { ref } from 'vue';
import axios from 'axios';

interface Pokemon {
  name: string;
  sprites: {
    front_default: string;
  };
}

// Array reativo para armazenar os Pokémon
const pokemons = ref<Pokemon[]>([]);

// Função para buscar os primeiros 151 Pokémon
const fetchPokemons = async () => {
  try {
    // Cria um array de promessas para os IDs de 1 a 151
    const requests = Array.from({ length: 151 }, (_, i) =>
      axios.get(`https://pokeapi.co/api/v2/pokemon/${i + 1}`)
    );

    // Aguarda a conclusão de todas as requisições
    const responses = await Promise.all(requests);

    // Extrai os dados relevantes de cada resposta
    pokemons.value = responses.map((response) => ({
      name: response.data.name,
      sprites: response.data.sprites,
    }));

    console.log(pokemons.value);
  } catch (error) {
    console.error('Erro ao buscar os Pokémon:', error);
  }
};

fetchPokemons();
</script>
