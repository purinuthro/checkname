<template>
    <div class="container">
      <header class="header">
        <div class="logo">
          <img src="/public/image/logo.jpg" alt="Logo">
        </div>
        <h1>เช็คชื่อนักเรียนโรงเรียนเทศบาล๔ (วัดโพธาวาส)</h1>
        <a href="/login" class="btn">Login</a>
      </header>
    </div>
  </template>
  
<script setup>
</script>

<style scoped>

@import url('https://fonts.googleapis.com/css2?family=Prompt:wght@400;700&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Prompt', sans-serif;
  background-color: #f4f4f4;
}

.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: linear-gradient(135deg, #df9aff 0%, #faeec4 100%);
  overflow: hidden;
}

.header {
  text-align: center;
  background: rgba(255, 255, 255, 0.9);
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.logo img {
  width: 120px;
  height: auto;
  margin-bottom: 20px;
}

.header h1 {
  font-size: 2rem;
  font-weight: 700;
  color: #333;
  margin-bottom: 20px;
}

.btn {
  display: inline-block;
  padding: 10px 30px;
  background-color: #6c63ff;
  color: #fff;
  font-size: 1rem;
  font-weight: 600;
  text-decoration: none;
  border-radius: 25px;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

.btn:hover {
  background-color: #5a54e6;
  transform: translateY(-3px);
}
</style>
