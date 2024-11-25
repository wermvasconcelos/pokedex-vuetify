<template>
  <v-app>
    <v-container>
      <h1>Pokédex - Vuetify</h1>
      <v-row>
        <!-- Itera sobre a lista de Pokémon e exibe cada um -->
        <v-col v-for="pokemon in pokemons" :key="pokemon.name" cols="12" sm="6" md="2">
          <v-card @click="openDialog(pokemon)">
            <v-card-title>{{ pokemon.name }}</v-card-title>
            <v-img :src="pokemon.sprites.front_default" alt="{{ pokemon.name }}" height="150" />
          </v-card>
        </v-col>
      </v-row>

      <!-- Modal de Detalhes do Pokémon -->
      <v-dialog v-model="dialog" max-width="500px" >
        <v-card>
          <v-card-title>
            <span class="headline">{{ selectedPokemon?.name }}</span>
          </v-card-title>
          <v-card-subtitle>Detalhes do Pokémon</v-card-subtitle>
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

interface Pokemon {
  name: string;
  sprites: {
    front_default: string;
  };
  sprites_gif: {
    front_default: string;
  }
}

// Array reativo para armazenar os Pokémon
const pokemons = ref<Pokemon[]>([]);

// Estado para controlar a exibição do modal
const dialog = ref(false);
const selectedPokemon = ref<Pokemon | null>(null);

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
      sprites_gif: response.data.sprites.other.showdown,
    }));

    console.log(pokemons.value);
  } catch (error) {
    console.error('Erro ao buscar os Pokémon:', error);
  }
};

// Função para abrir o modal com as informações do Pokémon selecionado
const openDialog = (pokemon: Pokemon) => {
  selectedPokemon.value = pokemon;
  dialog.value = true;
};

// Chama a função ao montar o componente
fetchPokemons();
</script>
