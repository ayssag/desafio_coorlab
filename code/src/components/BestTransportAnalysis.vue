<template>
    <div style="margin: 25px 25px 25px 25px; text-align: center;">
        <p>Digite o peso em kg</p>
        <input type="text" v-model="weight" placeholder="(sem unidade de medida)">
        <hr>

        <p>Escolha a cidade de destino</p>
        <b-form-select v-model="destiny" :options="cities"></b-form-select>
        <br>
        
        <br>
        <button @click="onSubmit">Analisar</button>
    </div>
</template>

<script>
export default {
    name: 'BestTransportAnalysis',
    props: ['cities'],
    data() {
        return {
            weight: '',
            destiny: ''
        };
    },
    methods: {
        onSubmit() {
            this.$emit('userInput', {
                'weight': Number(this.weight),
                'destiny': String(this.destiny)
            });
        }

    }
};
</script>