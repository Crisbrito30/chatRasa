<template>
    <div class="action-node" :class="{ selected: selected }">
      <Handle type="target" position="top" />
      <div class="content">
        <div class="header">
          <span class="icon">⚡</span>
          <span class="title">Action</span>
        </div>
        <div class="body">
          <input
            type="text"
            v-model="data.actionName"
            placeholder="Enter action name"
            @change="updateNode"
          />
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
  .action-node {
    background: #fff;
    border: 1px solid #1a192b;
    border-radius: 8px;
    padding: 10px;
    min-width: 150px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }
  
  .action-node.selected {
    border: 2px solid #ff9f1c;
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
  
  .body input {
    width: 100%;
    padding: 4px 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 0.9em;
  }
  
  .body input:focus {
    outline: none;
    border-color: #ff9f1c;
  }
  </style>