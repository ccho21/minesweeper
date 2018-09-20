<template>
    <div class="container">
        <div id="app" v-if="!start">
            <img src='./assets/logo.png' alt="Logo">
            <h1>{{ msg }}</h1>
            <button class="btn btn-default btn-lg" @click="playGame">Click to Play</button>
        </div>
        <div class="container">
            <router-view v-if="start"></router-view>
        </div>

    </div>

</template>

<script>
    export default {
        name: 'app',
        data() {
            return {
                msg: 'Welcome to Mind Sweeper',
                start : false
            }
        },
        methods: {
            playGame() {
                console.log("play game is working");
                this.$router.push({name: 'MineSweeper'});
                this.start = true;
            },
        }
    }
</script>

<style lang="scss">
    @import '../node_modules/bootstrap/scss/bootstrap.scss';
</style>
<style lang="scss">
    #app {
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        margin-top: 60px;
    }

    h1, h2 {
        font-weight: normal;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }
</style>
