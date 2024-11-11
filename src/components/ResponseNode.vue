<template>
    <div class="response-node" :class="{ selected: selected }">
      <Handle type="target" position="top" />
      <div class="content">
        <div class="header">
          <span class="icon">💬</span>
          <span class="title">Response</span>
        </div>
        <div class="body">
          <textarea
            v-model="data.response"
            placeholder="Enter bot response"
            @change="updateNode"
          ></textarea>
        </div>
      </div>
      <Handle type="source" position="bottom" />
    </div>
  </template>
  
  <script setup>
  import { Handle } from '@vue-flow/core'
  import { computed } from 'vue'
  
  const props = defineProps({
    id: {
      type: String,
      required: true
    },
    data: {
      type: Object,
      required: true
    },
    selected: {
      type: Boolean,
      default: false
    }
  })
  
  const emit = defineEmits(['update'])
  
  const updateNode = () => {
    emit('update', {
      id: props.id,
      data: props.data
    })
  }
  </script>
  
  <style scoped>
  .response-node {
    background: #fff;
    border: 1px solid #1a192b;
    border-radius: 8px;
    padding: 10px;
    min-width: 150px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }
  
  .response-node.selected {
    border: 2px solid #2ec4b6;
  }
  
  .header {
    display: flex;
    align-items: center;
    gap: 8px;
    margin-bottom: 8px;
  }
  
  .icon {
    font-size: 1.2em;
  }
  
  .title {
    font-weight: bold;
    color: #1a192b;
  }
  
  .body textarea {
    width: 100%;
    padding: 4px 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 0.9em;
    min-height: 60px;
    resize: vertical;
  }
  
  .body textarea:focus {
    outline: none;
    border-color: #2ec4b6;
  }
  </style>