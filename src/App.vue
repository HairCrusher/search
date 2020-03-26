<template>
    <div id="app">
        <button @click="generate" :disabled="loading">GENERATE DATA</button>
        <p v-if="loading">{{counter}}/{{strCount / chankLength}}</p>
        <hr>
        <input type="text" v-model="input"/><button @click="find">FIND</button>
        <p><span v-if="searchLoading">Loading...</span><span v-else>Result: {{result}}</span></p>
    </div>
</template>

<script>
    import {randString} from './helpers';
    import localforage from 'localforage';

    export default {
        name: 'App',
        data() {
            return {
                loading: false,
                searchLoading: false,
                counter: 0,
                input: '',
                result: 0,
                strCount: 10**7,
                chankLength: 10**6
            };
        },
        methods: {
            async generate() {
                console.log('START AT:'+ new Date());
                this.loading = true;

                await localforage.setDriver(localforage.INDEXEDDB);
                await localforage.clear();

                let str = '';

                let j = 0;
                for (let i = 0; i < this.strCount; i++) {
                    str += randString(100) + ':';

                    if (!(i % this.chankLength)) {
                        await localforage.setItem(j.toString(), str);
                        this.counter++;
                        j++;
                        str = '';
                    }
                }
                this.loading = false;
                console.log('END AT:'+ new Date());
            },
            async find() {
                await localforage.setDriver(localforage.INDEXEDDB);

                this.result = 0;
                this.searchLoading = true;
                console.log('START', new Date());
                const reg = new RegExp(`(^|\\:)${this.input}`, 'g');
                for (let i = 0; i < this.strCount / this.chankLength; i++){
                    const str = await localforage.getItem(i.toString());
                    // console.log(new Date(), i);
                    const res = str.match(reg);
                    console.log(`Strings [${i * this.chankLength} - ${(i+1) * this.chankLength}] match - ${res ? res.length : 0}|`, res);
                    if(res) this.result += res.length;
                }
                console.log('END', new Date());
                this.searchLoading = false;
            }
        }
    }
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
