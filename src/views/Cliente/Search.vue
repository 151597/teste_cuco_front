<template>
    <div>
        <v-row justify="space-between">
            <v-col cols="3" md="3" lg="2" xs="12">
                <v-select
                :items="itemsPerPageProp"
                v-model="perSelectedProp"
                 @input="$emit('perSelectedProp', perSelectedProp)"
                label="Items por Página"
                ></v-select>
            </v-col>
            <v-col cols="6" md="3" lg="3" xs="12">
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
