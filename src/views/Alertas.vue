<template>
    <div>
      <h1>Lista de Alertas</h1>
  
      <!-- Lista de Alertas -->
      <ul v-if="alertas.length">
        <li v-for="alerta in alertas" :key="alerta.id">
          <p><strong>ID:</strong> {{ alerta.id }}</p>
          <p><strong>Termo:</strong> {{ alerta.mensagem }}</p>
          <p><strong>Significado:</strong> {{ alerta.detalhe }}</p>
          <p><strong>Versão:</strong> {{ alerta.nivel }}</p>
          <p>
            <strong>Situação:</strong>
            {{ alerta.nivel >= 3 ? "Crítica" : "Controlada" }}
          </p>
        </li>
      </ul>
      <p v-else>Nenhum alerta disponível.</p>
  
      <!-- Cadastro de Novo Alerta -->
      <h2>Cadastrar Alerta</h2>
      <form @submit.prevent="cadastrarAlerta">
        <label>
          Mensagem:
          <input v-model="novoAlerta.mensagem" required />
        </label>
        <label>
          Detalhe:
          <input v-model="novoAlerta.detalhe" />
        </label>
        <label>
          Data/Hora Geração:
          <input v-model="novoAlerta.dataHoraGeracao" type="datetime-local" />
        </label>
        <label>
          Data/Hora Verificação:
          <input v-model="novoAlerta.dataHoraVerificacao" type="datetime-local" />
        </label>
        <label>
          Nível:
          <input v-model.number="novoAlerta.nivel" type="number" />
        </label>
        <button type="submit">Cadastrar</button>
      </form>
  
      <!-- Busca por Alerta -->
      <h2>Buscar Alertas</h2>
      <form @submit.prevent="buscarAlertas">
        <label>
          Data/Hora Verificação:
          <input v-model="filtros.dataHora" type="datetime-local" required />
        </label>
        <label>
          Mensagem:
          <input v-model="filtros.mensagem" />
        </label>
        <button type="submit">Buscar</button>
      </form>
  
      <!-- Mensagem de nenhum alerta encontrado -->
      <p v-if="exibeMensagem">Nenhum alerta foi encontrado para os critérios fornecidos.</p>
    </div>
  </template>
  
  <script setup>
  import { ref } from "vue";
  import axios from "axios";
  
  // Estados
  const alertas = ref([]);
  const novoAlerta = ref({
    mensagem: "",
    detalhe: "",
    dataHoraGeracao: "",
    dataHoraVerificacao: "",
    nivel: null,
  });
  const filtros = ref({ dataHora: "", mensagem: "" });
  const exibeMensagem = ref(false);
  
  // Função para buscar todos os alertas
  const carregarAlertas = async () => {
    try {
      const response = await axios.get("/alerta");
      alertas.value = response.data;
      exibeMensagem.value = false;
    } catch (error) {
      console.error("Erro ao carregar alertas:", error);
    }
  };
  
  // Função para cadastrar um alerta
  const cadastrarAlerta = async () => {
    try {
      if (!novoAlerta.value.mensagem) {
        alert("O campo 'Mensagem' é obrigatório.");
        return;
      }
      const response = await axios.post("/alerta", novoAlerta.value);
      alertas.value.push(response.data);
      // Limpa o formulário
      novoAlerta.value = {
        mensagem: "",
        detalhe: "",
        dataHoraGeracao: "",
        dataHoraVerificacao: "",
        nivel: null,
      };
    } catch (error) {
      console.error("Erro ao cadastrar alerta:", error);
    }
  };
  
  // Função para buscar alertas por data/hora e mensagem
  const buscarAlertas = async () => {
    try {
      const { dataHora, mensagem } = filtros.value;
      const response = await axios.get(
        `/alerta/buscaMensagem/${dataHora}/${mensagem}`
      );
      if (response.data.length) {
        alertas.value = response.data;
        exibeMensagem.value = false;
      } else {
        alertas.value = [];
        exibeMensagem.value = true;
      }
    } catch (error) {
      console.error("Erro ao buscar alertas:", error);
    }
  };
  
  // Inicialização
  carregarAlertas();
  </script>
  