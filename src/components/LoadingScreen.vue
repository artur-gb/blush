<template>
    <div class="overlay">
      <img src="../assets/logo1.gif" class="centered-image" />
    </div>
  </template>
  
  <style scoped>
  .overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.3);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 5;
  }
  
  .centered-image {
    max-width: 100%;
    max-height: 100%;
  }
  </style>