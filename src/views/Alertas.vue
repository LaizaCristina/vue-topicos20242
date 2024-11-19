<template>
    <div class="app-container">
      <h1>Lista de Alertas</h1>
  
      <!-- Tabela de Alertas -->
      <table v-if="alertas.length" class="alert-table">
        <thead>
          <tr>
            <th>Mensagem</th>
            <th>Detalhe</th>
            <th>Nível</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="alerta in alertas" :key="alerta.id">
            <td>{{ alerta.mensagem }}</td>
            <td>{{ alerta.detalhe }}</td>
            <td>{{ alerta.nivel }}</td>
          </tr>
        </tbody>
      </table>
      <p v-else>Nenhum alerta disponível.</p>
  
      <!-- Cadastro de Novo Alerta -->
      <h2>Cadastrar Alerta</h2>
      <form @submit.prevent="cadastrarAlerta" class="form-container">
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
          <input v-model="novoAlerta.dataHoraGeracao" type="datetime-local" required />
        </label>
        <label>
          Data/Hora Verificação:
          <input v-model="novoAlerta.dataHoraVerificacao" type="datetime-local" required />
        </label>
        <label>
          Nível:
          <input v-model.number="novoAlerta.nivel" type="number" required />
        </label>
        <button type="submit">Cadastrar</button>
      </form>
  
      <!-- Mensagem de erro se os campos obrigatórios não forem preenchidos -->
      <p v-if="erroCadastro" class="error-message">Por favor, preencha todos os campos obrigatórios.</p>
  
      <!-- Busca por Alerta -->
      <h2>Buscar Alertas</h2>
      <form @submit.prevent="buscarAlertas" class="form-container">
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
  import { ref } from 'vue';
  import axios from 'axios';
  
  // Estados
  const alertas = ref([]);
  const novoAlerta = ref({
    mensagem: '',
    detalhe: '',
    dataHoraGeracao: '',
    dataHoraVerificacao: '',
    nivel: null,
  });
  const filtros = ref({ dataHora: '', mensagem: '' });
  const exibeMensagem = ref(false);
  const erroCadastro = ref(false); 
  
  // Função para buscar alertas
  const buscarAlertas = async () => {
    try {
      const { dataHora, mensagem } = filtros.value;
  
      if (!dataHora) {
        alert("O campo 'Data/Hora Verificação' é obrigatório.");
        return;
      }
  
      const mensagemFormatada = mensagem || 'null';
  
      const response = await axios.get(
        `/alerta/buscaMensagem/${dataHora}/${mensagemFormatada}`
      );
  
      if (response.data.length) {
        alertas.value = response.data;
        exibeMensagem.value = false;
      } else {
        alertas.value = [];
        exibeMensagem.value = true;
      }
    } catch (error) {
      console.error('Erro ao buscar alertas:', error);
      alert('Erro ao realizar a busca. Verifique os critérios fornecidos.');
    }
  };
  
  // Função para cadastrar alerta
  const cadastrarAlerta = async () => {
  
  if (
    !novoAlerta.value.mensagem ||
    !novoAlerta.value.detalhe ||
    !novoAlerta.value.dataHoraGeracao ||
    !novoAlerta.value.dataHoraVerificacao ||
    novoAlerta.value.nivel === null
  ) {
    
    alert('Por favor, preencha todos os campos obrigatórios.');
    return;
  }

  try {
    const response = await axios.post('/alerta', novoAlerta.value);
    alertas.value.push(response.data);

   
    novoAlerta.value = {
      mensagem: '',
      detalhe: '',
      dataHoraGeracao: '',
      dataHoraVerificacao: '',
      nivel: null,
    };
  } catch (error) {
    console.error('Erro ao cadastrar alerta:', error);
  }
};

  
  // Função para carregar os alertas
  const carregarAlertas = async () => {
    try {
      const response = await axios.get('/alerta');
      alertas.value = response.data;
      exibeMensagem.value = false;
    } catch (error) {
      console.error('Erro ao carregar alertas:', error);
    }
  };
  
  carregarAlertas();
  </script>
  
  <style scoped>
  /* Estilos existentes */
  .error-message {
    color: red;
    font-weight: bold;
    margin-top: 10px;
  }
  
  .app-container {
    font-family: Arial, sans-serif;
    padding: 20px;
    background-color: #f4f9f9;
    border-radius: 10px;
    max-width: 900px;
    margin: 0 auto;
  }
  
  h1, h2 {
    color: #4caf50;
    text-align: center;
  }
  
  .form-container {
    display: grid;
    gap: 10px;
    margin-bottom: 20px;
  }
  
  label {
    font-size: 14px;
  }
  
  input {
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
  
  button {
    padding: 10px 15px;
    background-color: #4caf50;
    border: none;
    color: white;
    border-radius: 5px;
    cursor: pointer;
  }
  
  button:hover {
    background-color: #45a049;
  }
  
  .alert-table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
  }
  
  .alert-table th, .alert-table td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }
  
  .alert-table th {
    background-color: #4caf50;
    color: white;
  }
  
  .alert-table tr:nth-child(even) {
    background-color: #f2f2f2;
  }
  </style>
  