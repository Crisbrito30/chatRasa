<!-- RasaFlowEditor.vue -->
<template>
  <div class="rasa-flow-editor">
    <div class="toolbar">
      <button @click="addNode('intent')" class="btn">
        <i class="fas fa-comment"></i>
        Add Intent
      </button>
      <button @click="addNode('action')" class="btn">
        <i class="fas fa-robot"></i>
        Add Action
      </button>
      <button @click="addNode('response')" class="btn">
        <i class="fas fa-reply"></i>
        Add Response
      </button>
      <button @click="sendFlowToRasa(rasaConfig)" class="btn btn-primary">
  <i class="fas fa-paper-plane"></i>
  Enviar para o Rasa
</button>
    </div>

    <vue-flow v-model="elements" 
      :default-viewport="{ x: 0, y: 0, zoom: 1 }" @connect="onConnect"
      @nodeDragStop="onNodeDragStop" 
      class="flow-container">

    
      <template #node-action="actionProps">
        <action-node v-bind="actionProps" />
        <button @click="removeNode(actionProps.id)">Excluir</button>
      </template>
      <template #node-response="responseProps">
        <response-node v-bind="responseProps" />
        <button @click="removeNode(responseProps.id)">Excluir</button>
      </template>
      <template #node-intent="intentProps">
        <intent-node v-bind="intentProps" />
        <button @click="removeNode(intentProps.id)">Excluir</button>
      </template>
      <template #panel>
        <vue-flow-controls />
        <vue-flow-mini-map />
      </template>

      <background pattern-color="#aaa" gap="8" />
    </vue-flow>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { VueFlow, } from '@vue-flow/core'
import '@vue-flow/core/dist/style.css'
import '@vue-flow/controls/dist/style.css'
import '@vue-flow/minimap/dist/style.css'
import IntentNode from './IntentNode.vue'
import ActionNode from './ActionNode.vue'
import ResponseNode from './ResponseNode.vue'
import axios from 'axios';

// Estado inicial
const elements = ref([
  {
    id: '1',
    type: 'intent',
    label: 'greet',
    position: { x: 100, y: 100 },
    data: { examples: ['hello', 'hi'] }
  },
  {
    id: '2',
    type: 'response',
    label: 'utter_greet',
    position: { x: 300, y: 100 },
    data: { text: 'Hello! How can I help?' }
  },
  {
    id: 'e1-2',
    source: '1',
    target: '2'
  }
])

// Função para adicionar um novo node
const addNode = (type) => {
  const newNode = {
    id: `node-${elements.value.length + 1}`,
    type,
    label: `Novo ${type.charAt(0).toUpperCase() + type.slice(1)}`,
    position: { x: Math.random() * 400, y: Math.random() * 400 },
    data: {
      examples: type === 'intent' ? [] : undefined,
      text: type === 'response' ? '' : undefined
    }
  };
  elements.value.push(newNode);
  console.log('Node adicionado:', newNode);
};

// Função para exportar o fluxo no formato Rasa
const exportToRasa = () => {
  const rasaConfig = {
    intents: [],
    actions: [],
    responses: {},
    stories: []
  };

   // Coletar dados dos nós
   elements.value.forEach(el => {
    if (el.type === 'intent') {
      rasaConfig.intents.push({
        name: el.label,
        examples: el.data.examples
      });
    } else if (el.type === 'action') {
      rasaConfig.actions.push(el.label);
    } else if (el.type === 'response') {
      rasaConfig.responses[el.label] = [{ text: el.data.text }];
    }
  });

 // Coletar dados das conexões para criar as histórias
 const edges = elements.value.filter(el => el.source && el.target);
  edges.forEach(edge => {
    const sourceNode = elements.value.find(n => n.id === edge.source);
    const targetNode = elements.value.find(n => n.id === edge.target);

    if (sourceNode && targetNode) {
      rasaConfig.stories.push({
        story: `${sourceNode.label}_to_${targetNode.label}`,
        steps: [
          { intent: sourceNode.label },
          { action: targetNode.label }
        ]
      });
    }
  });

  return rasaConfig;
};

// Função para enviar o fluxo para o Rasa
const sendFlowToRasa = async (flowData) => {
  try {
    const response = await axios.post('http://localhost:5005/webhooks/rest/webhook', flowData, {
      headers: { 'Content-Type': 'application/json' }
    });
    console.log('Flow sent successfully:', response.data);
  } catch (error) {
    console.error('Error sending flow to Rasa:', error);
  }
};
// Manipular conexões entre nós
const onConnect = (params) => {
  const isDuplicate = elements.value.some(
    el => el.source === params.source && el.target === params.target
  );
  if (!isDuplicate) {
    elements.value.push({
      id: `edge-${Date.now()}`,
      source: params.source,
      target: params.target
    });
  }
};

// Função para remover um nó e suas conexões
const removeNode = (nodeId) => {
  // Remover o nó de `elements` (intent, action, response)
  elements.value = elements.value.filter(element => element.id !== nodeId);

  // Remover as conexões associadas ao nó
  elements.value = elements.value.filter(element => element.source !== nodeId && element.target !== nodeId);

  // Se houver ações ou respostas associadas a esse nó, também serão removidas.
  elements.value.forEach(el => {
    if (el.type === 'action' || el.type === 'response') {
      if (el.source === nodeId || el.target === nodeId) {
        elements.value = elements.value.filter(element => element.id !== el.id);
      }
    }
  });
};

</script>

<style scoped>
.rasa-flow-editor {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.toolbar {
  padding: 1rem;
  border-bottom: 1px solid #e2e8f0;
  display: flex;
  gap: 1rem;
}

/* Use a purple theme for our custom node */
.vue-flow__node-custom {
  background: purple;
  color: white;
  border: 1px solid purple;
  border-radius: 4px;
  box-shadow: 0 0 0 1px purple;
  padding: 8px;
}

.flow-container {
  flex: 1;
  background: #f8fafc;
}

.btn {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  border: 1px solid #e2e8f0;
  border-radius: 0.375rem;
  background: white;
  cursor: pointer;
  transition: all 0.2s;
}

.btn:hover {
  background: #f1f5f9;
}

.btn-primary {
  background: #3b82f6;
  color: white;
  border-color: #2563eb;
}

.btn-primary:hover {
  background: #2563eb;
}
</style>
