<template>
  <div id="app">
    <el-container>
      <el-header> Event Scheduler </el-header>
      <el-main>
        <EventForm msg="Welcome to Your Vue.js App" />
      </el-main>
      <el-footer>@2023</el-footer>
    </el-container>
  </div>
</template>

<script>
import EventForm from './components/EventForm.vue';

export default {
  name: 'App',
  components: {
    EventForm,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
