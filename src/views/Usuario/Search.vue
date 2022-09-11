<template>
    <div>
        <v-row justify="end">
            <v-col cols="12" md="3" lg="3" xs="12">
                <v-text-field
                    label="Pesquisa"
                    v-model="searchProp"
                    @input="$emit('searchProp', searchProp)"
                    prepend-inner-icon="mdi-magnify"
                    hide-details
                    >
                </v-text-field>
            </v-col>
        </v-row>
    </div>
</template>

<script>
    export default {
        name: 'Search',
        props: ['search','statusSelected', 'status', 'contratoSelected', 'contrato', 'perSelected', 'itemsPerPage'],
        data(){
            return {
                searchProp: this.search,
                perSelectedProp: this.perSelected,
                itemsPerPageProp: this.itemsPerPage
            }
        }
    }
</script>

<style scoped>

</style>
