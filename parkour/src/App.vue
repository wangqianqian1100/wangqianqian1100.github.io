<template>
  <div id="app">
    <img src="./assets/logo.png">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50; 
  height: 100%;
  width: 100%;
}
body,h1,h3,h2,h4,h5,h6,p,dl,dd,ul,ol,form,input,fieldset,th,td,iframe{margin:0; padding:0;}
h1,h2,h3,h4,h5,h6,strong,b{font-weight:normal;}
i,em{font-style:normal;}
a{text-decoration:none; color:#666;}
li{list-style:none;}
img{border:none;vertical-align:top;pointer-events:none;}
input{vertical-align:top;}
a,input,area{outline:none;}
body{font-family:"黑体";}
html,body{width: 100%;height: 100%;}
*{-webkit-tap-highlight-color: rgba(0,0,0,0);}
</style>
