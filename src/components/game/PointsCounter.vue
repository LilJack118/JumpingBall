<template>
    <div class="counter-box" v-bind:class="counterColorClass">
        <h1 class="counter">{{$store.getters['counter']}}</h1>
    </div>
</template>

<script>
export default{
    computed:{
    counterColorClass(){
      return this.$store.getters['getColor'].counter;
    },
    }
}
</script>

<style scoped>
.counter-box{
    position: absolute;
    width: 40%;
    aspect-ratio:1/1;
    border-radius:50%;
    display:flex;
    text-align: center;
    left: 0; right: 0; top: 0; bottom: 0;
    margin: auto;
    /* background: #343A40; */
}
.counter{
    -webkit-touch-callout: none; /* iOS Safari */
    -webkit-user-select: none; /* Safari */
    -khtml-user-select: none; /* Konqueror HTML */
    -moz-user-select: none; /* Old versions of Firefox */
    -ms-user-select: none; /* Internet Explorer/Edge */
    user-select: none; /* Non-prefixed version, currently
                            supported by Chrome, Edge, Opera and Firefox */
    font-size:80px;
    /* color: #343A40; */
    margin:auto;
    font-weight: 900;
}
</style>