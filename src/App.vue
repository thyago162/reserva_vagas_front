<script setup>
import { ref, onMounted, nextTick } from 'vue'
import axios from 'axios'

const mensagemAtual = ref('')
const historicoChat = ref([
  { role: 'assistant', content: 'Olá! Sou o seu assistente de atendimento escolar. Como posso ajudar com as vagas e reservas hoje?' }
])
const carregando = ref(false)
const chatContainer = ref(null)
const textareaRef = ref(null)

const sessionId = ref('')

onMounted(() => {
  let storedSession = localStorage.getItem('chat_session_id')
  if (!storedSession) {
    storedSession = 'sessao-' + Math.random().toString(36).substring(2, 15)
    localStorage.setItem('chat_session_id', storedSession)
  }
  sessionId.value = storedSession
})

const rolarParaBaixo = async () => {
  await nextTick()
  if (chatContainer.value) {
    chatContainer.value.scrollTop = chatContainer.value.scrollHeight
  }
}

const enviarMensagem = async () => {
  const texto = mensagemAtual.value.trim()
  if (!texto || carregando.value) return

  historicoChat.value.push({ role: 'user', content: texto })
  mensagemAtual.value = ''
  carregando.value = true
  await rolarParaBaixo()

  try {
    const resposta = await axios.post('http://localhost:8000/chat', {
      message: texto,
      session_id: sessionId.value
    })

    historicoChat.value.push({
      role: 'assistant',
      content: resposta.data.response
    })

  } catch (erro) {
    console.error('Erro ao comunicar com o agente:', erro)
    historicoChat.value.push({
      role: 'assistant',
      content: '❌ Desculpe, ocorreu um erro ao processar a sua solicitação. Tente novamente mais tarde.'
    })
  } finally {
    carregando.value = false
    await rolarParaBaixo()
  }
}

const lidarComTecla = (e) => {
  if (e.key === 'Enter' && !e.shiftKey) {
    e.preventDefault()
    enviarMensagem()
  }
}
</script>

<template>
  <div class="chat-wrapper">
    <header class="chat-header">
      <div class="header-top">
        <h2>🤖 Assistente de Vagas Escolares</h2>
        <span class="session-badge">Sessão: {{ sessionId.substring(0, 8) }}...</span>
      </div>
      
      <div class="capabilities-box">
        <h4>O que posso fazer:</h4>
        <ul>
          <li>Consultar Reservas</li>
          <li>Buscar escolas</li>
          <li>Fazer reservas</li>
        </ul>
      </div>
    </header>

    <div class="chat-messages" ref="chatContainer">
      <div 
        v-for="(msg, index) in historicoChat" 
        :key="index" 
        :class="['message-bubble', msg.role === 'user' ? 'user-msg' : 'assistant-msg']"
      >
        <div class="message-content">
          <p style="white-space: pre-wrap; margin: 0;">{{ msg.content }}</p>
        </div>
      </div>

      <div v-if="carregando" class="message-bubble assistant-msg">
        <div class="message-content typing">
          <span>.</span><span>.</span><span>.</span>
        </div>
      </div>
    </div>

    <!-- Área do input com rows="1" correto -->
    <div class="chat-input-area">
      <textarea 
        ref="textareaRef"
        v-model="mensagemAtual" 
        @keydown="lidarComTecla"
        placeholder="Escreva a sua dúvida ou pedido de reserva... (Enter para enviar, Shift+Enter para nova linha)" 
        :disabled="carregando"
        rows="1"
      ></textarea>
      <button @click="enviarMensagem" :disabled="carregando">Enviar</button>
    </div>
  </div>
</template>

<style scoped>
.chat-wrapper {
  max-width: 750px;
  height: 90vh;
  margin: 20px auto;
  display: flex;
  flex-direction: column;
  border: 1px solid #ddd;
  border-radius: 12px;
  background: #f9f9f9;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  font-family: Arial, sans-serif;
}

.chat-header {
  padding: 16px;
  background: #2c3e50;
  color: white;
  border-top-left-radius: 12px;
  border-top-right-radius: 12px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.header-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.chat-header h2 {
  margin: 0;
  font-size: 1.2rem;
}

.capabilities-box {
  background: rgba(255, 255, 255, 0.1);
  padding: 8px 12px;
  border-radius: 6px;
}

.capabilities-box h4 {
  margin: 0 0 4px 0;
  font-size: 0.85rem;
  color: #ecf0f1;
}

.capabilities-box ul {
  margin: 0;
  padding-left: 18px;
  font-size: 0.85rem;
  display: flex;
  gap: 15px;
  list-style-type: disc;
}

.session-badge {
  font-size: 11px;
  background: rgba(255,255,255,0.2);
  padding: 4px 8px;
  border-radius: 4px;
}

.chat-messages {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.message-bubble {
  max-width: 75%;
  padding: 12px 16px;
  border-radius: 12px;
  line-height: 1.4;
}

.user-msg {
  background: #3498db;
  color: white;
  align-self: flex-end;
  border-bottom-right-radius: 2px;
}

.assistant-msg {
  background: #e1e8ed;
  color: #333;
  align-self: flex-start;
  border-bottom-left-radius: 2px;
}

.chat-input-area {
  padding: 16px;
  background: white;
  border-top: 1px solid #ddd;
  display: flex;
  align-items: flex-end;
  gap: 10px;
  border-bottom-left-radius: 12px;
  border-bottom-right-radius: 12px;
}

.chat-input-area textarea {
  flex: 1;
  padding: 10px 14px;
  border: 1px solid #ccc;
  border-radius: 6px;
  outline: none;
  font-size: 14px;
  resize: none;
  max-height: 120px;
  font-family: inherit;
  line-height: 1.4;
}

.chat-input-area button {
  padding: 10px 20px;
  background: #2ecc71;
  color: white;
  border: none;
  border-radius: 6px;
  font-weight: bold;
  cursor: pointer;
  height: 41px;
}

.chat-input-area button:disabled {
  background: #95a5a6;
  cursor: not-allowed;
}

.typing span {
  animation: blink 1.4s infinite both;
  font-size: 24px;
  font-weight: bold;
}
.typing span:nth-child(2) { animation-delay: .2s; }
.typing span:nth-child(3) { animation-delay: .4s; }

@keyframes blink {
  0% { opacity: .2; }
  20% { opacity: 1; }
  100% { opacity: .2; }
}
</style>