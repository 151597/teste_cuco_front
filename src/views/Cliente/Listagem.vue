<template>
    <!-- DATATABLE -->
    <v-row>
        <v-col cols="12" md="12" sm="12" xs="12">
                <v-data-table
                        :search="search"
                        no-results-text="Nenhum resultado encontrado!"
                        loading-text="Carregando... Por Favor aguarde"
                        :headers="headers"
                        :items="rows"
                        :page.sync="page"
                        :items-per-page="perSelected.value"
                        hide-default-footer
                        class="elevation-1"
                        @page-count="pageCount = $event">
                                <template v-slot:[`item.actions`]="{ item }">
                                        <v-btn icon @click="loadClient(item)"><v-icon>mdi-pencil</v-icon></v-btn>
                                        <v-btn icon @click="remove(item)"><v-icon>mdi-delete</v-icon></v-btn>
                                </template>
                </v-data-table>
                <div class="text-center-pt-2">
                        <v-pagination
                                v-model="page"
                                :length="pageCount">
                        </v-pagination>
                </div>
        </v-col>
    </v-row>
</template>

<script>
import { baseApiUrl } from '@/global'
import axios from 'axios'
import Swal from 'sweetalert2'
export default {
    name: 'ClienteListagem',
    props: [ 'search', 'dialog', 'client'],
    data(){
        return {
            filters: { 'status': [] },
			activeFilters: {},
			perSelected: {text: 10, value: 10},
			itemsPerPage: [
					{text: 5, value: 5},
					{text: 10, value: 10},
					{text: 15, value: 15},
					{text: 'All', value: -1}
			],
			page: 1,
			pageCount: 0,
			headers: [
					{
						text: 'Cliente',
						align: 'start',
						sortable: false,
						value: 'nome'
					},
					{ text: 'CPF', value: 'cpf', sortable: true },
					{ text: 'Data de Nascimento', value: 'data_nascimento', sortable: true },
					{ text: 'Telefone', value: 'telefone', sortable: true },
					{ text: 'Ações', value: 'actions' }
			],
			rows: []
        }
    },
    watch: {
        //cria as linhas da tabela
        rows(){
			this.initFilters()
		}
    },
    methods:{
        // inicializa a opção de busca na tabela
        initFilters() {
            for (var col in this.filters) {
                this.filters[col] = this.rows.map((d) => { return d[col] }).filter(
                        (value, index, self) => { return self.indexOf(value) === index }
                )
            }
            // TODO restore previous activeFilters before add/remove item
            this.activeFilters = Object.assign({}, this.filters)
		},
		loadClients(){
            const url = `${baseApiUrl}/clients`
            axios.get(url).then(res => {
                this.clients = res.data
				this.rows = res.data
            })
        },
        // faz o reload dos clientes
		resetClient(){
            this.loadClients()
        },
        // exclui cliente com base no id
		remove(remove){
            const id = remove.id
            Swal.fire({
                title: 'Tem certeza que deseja excluir?',
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Sim!',
                cancelButtonText: 'Cancelar'
            }).then((result) => {
                if (result.isConfirmed) {
                    axios.delete(`${baseApiUrl}/clients/${id}`)
                        .then(() => {
                            Swal.fire('Excluido!', '', 'success')
                            this.resetClient()
                        })
                        .catch(() => {
                            Swal.fire({
                                icon: 'error',
                                title: 'Oops...',
                                text: 'Contate o Administrador!',
                            })
                        })
                }
            })
        },
        //carrega os valores no modal emitindo um evento para o pai component
		loadClient(client){
			console.log(client)
			this.panel = 0
			this.isDisabled = false
            this.$emit('update-dialog', { ...client })
        }
    },
    mounted(){
        this.loadClients()
    }
}
</script>

<style>

</style>