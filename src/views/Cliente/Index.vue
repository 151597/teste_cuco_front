<template>
	<div class="pa-5">
		<v-card cardContainer>
			<v-card-text>
				<v-row>
					<v-col cols="auto">
						<v-dialog
							max-width="600"
							persistent
							v-model="dialog"
						>
							<template v-slot:activator="{ on, attrs }">
							<v-btn
								color="primary"
								v-bind="attrs"
								v-on="on"
								@click="resetClient"
							>Criar</v-btn>
							</template>
							<template>
							<v-card>
								<v-toolbar
								color="primary"
								dark
								>Criar</v-toolbar>
								<v-card-text>
									<v-row>
										<v-col cols="12" md="12" sm="12" xs="12">
											<v-form ref="form" @submit.prevent="submit">
												<input type="hidden" v-model="client.id">
												<v-row>
													<v-col cols="12" md="6" sm="12" xs="12">
															<v-text-field prepend-icon="mdi-account" label="Nome Completo" name="nome" type="text" v-model="client.nome" :rules="rules"></v-text-field>
													</v-col>
													<v-col cols="12" md="6" sm="12" xs="12">
															<v-text-field label="CPF" name="cpf" type="cpf" v-model="client.cpf" 
															prepend-icon="mdi-card-account-details" v-mask="'###.###.###-##'" :rules="rulesCpf"></v-text-field>
													</v-col>
												</v-row>
												<v-row>
													<v-col cols="12" md="6" sm="12" xs="12">
														<v-menu
															ref="menu"
															v-model="menu"
															:close-on-content-click="false"
															transition="scale-transition"
															offset-y
															min-width="auto"
															>
															<template v-slot:activator="{ on, attrs }">
																<v-text-field
																v-model="client.dateFormatted"
																label="Data de Nascimento"
																prepend-icon="mdi-calendar"
																readonly
																v-bind="attrs"
																v-on="on"
																></v-text-field>
															</template>
															<v-date-picker
																v-model="date"
																locale="pt-br"
																:active-picker.sync="activePicker"
																:max="(new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10)"
																min="1950-01-01"
																@change="saveDate"
															></v-date-picker>
														</v-menu>
													</v-col>
													<v-col cols="12" md="6" sm="12" xs="12">
														<v-text-field ref="telefone" prepend-icon="mdi-phone" label="Telefone" 
															name="telefone" type="text" v-mask="phoneNumberMask"
															v-model="client.telefone" :rules="rulesTelefone"></v-text-field>
													</v-col>
												</v-row>
												<v-row>
													<v-btn @click="saveClient" :disabled="isDisabled" color="primary">Salvar</v-btn>
												</v-row>
											</v-form>
										</v-col>
									</v-row>
								</v-card-text>
								<v-card-actions class="justify-end">
								<v-btn
									text
									@click="resetClient"
								>Fechar</v-btn>
								</v-card-actions>
							</v-card>
							</template>
						</v-dialog>
					</v-col>
				</v-row>
				<Search
				:search="search"
				@searchProp="search = $event"
				@perSelectedProp="perSelectedProp = parseInt($event, 10)"
				:itemsPerPage="itemsPerPage"/>
				<Listagem ref="listagem" :search="search" :perSelectedProp="perSelectedProp" :dialog="dialog" :client="client" @update-dialog="updateDialog($event)"/>
			</v-card-text>
		</v-card>
	</div>
</template>

<script>
import Search from './Search.vue'
import Listagem from './Listagem.vue'
import { baseApiUrl } from '@/global'
import axios from 'axios'
import moment from 'moment'
import Swal from 'sweetalert2'
export default {
	nome: 'ClienteIndex',
	components: { Search, Listagem },
	data(){
		return {
			mode: 'saveClient',
			dialog: false,
			panel: 1,
			showPassword: false,
			isDisabled: true,
			client: {},
			rules: [
				value => !!value || 'Obrigatório.'
			],
			rulesCpf: [
				value => !!value || 'Obrigatório.',
				value => this.validarCPF(value) || 'CPF inválido'
			],
			rulesTelefone: [
				value => (value || '').length >= 15 || 'Número inválido'
			],
			activePicker: null,
			menu: false,
			date: '',
			search: '',
			perSelectedProp: 5,
			itemsPerPage: [
				{text: 5, value: 5},
				{text: 10, value: 10},
				{text: 15, value: 15},
				{text: 'All', value: -1}
			],
			phoneNumberMask: '',
			changeCpf: false,
			oldCpf: ''
		}
	},
	watch: {
		//menu do datepicker
		menu (val) {
			val && setTimeout(() => (this.activePicker = 'YEAR'))
		},
		//verifica se o campo cpf foi preenchido e é valido
		date () {
			this.client.dateFormatted = this.formatDate(this.date)
			if(this.validarCPF(this.client.cpf) && this.client.cpf != "" && this.client.nome != "" && this.date != ""){
				this.isDisabled = false
			}else{
				this.isDisabled = true
			}
		},
		client: {
			handler(val){
				if(val.cpf && val.nome && val.dateFormatted){
					if(this.validarCPF(val.cpf) && val.cpf != "" && val.nome != "" && val.dateFormatted != ""){
						this.isDisabled = false
					}else{
						this.isDisabled = true
					}
				}else{
					this.isDisabled = true
				}
			},
			deep: true
		},
		//verifica se o campo de cpf mudou
		'client.cpf'(newVal){
			if(newVal && this.oldCpf){
				if(newVal.replace(/[^\d]/g, "") !== this.oldCpf.replace(/[^\d]/g, "")){
					this.changeCpf = true
				}else{
					this.changeCpf = false
				}
			}
		},
    },
    methods: {
		//salva a data no campo datepicker
		saveDate (date) {
			this.$refs.menu.save(date)
		},
		//formata a data para o padrão pt-BR
		formatDate (date) {
			if (!date) return null

			const [year, month, day] = date.split('-')
			return `${day}/${month}/${year}`
		},
		validarCPF(cpf) {
			if(cpf){
				cpf = cpf.replace(/\./g, '');
				cpf = cpf.replace('-', '');
				cpf = cpf.split('');
			}
			if(cpf && cpf.length === 11){
				
				
				let v1 = 0;
				let v2 = 0;
				let aux = false;
				
				for (let i = 1; cpf.length > i; i++) {
					if (cpf[i - 1] != cpf[i]) {
						aux = true;   
					}
				} 
				
				if (aux == false) {
					return false; 
				} 
				
				for (let i = 0, p = 10; (cpf.length - 2) > i; i++, p--) {
					v1 += cpf[i] * p; 
				} 
				
				v1 = ((v1 * 10) % 11);
				
				if (v1 == 10) {
					v1 = 0; 
				}
				
				if (v1 != cpf[9]) {
					return false; 
				} 
				
				for (let i = 0, p = 11; (cpf.length - 1) > i; i++, p--) {
					v2 += cpf[i] * p; 
				} 
				
				v2 = ((v2 * 10) % 11);
				
				if (v2 == 10) {
					v2 = 0; 
				}
				
				if (v2 != cpf[10]) {
					return false; 
				} else {		
					return true;
				}
			}else{
				return false
			}
		},
		//resetar os dados
		resetClient(){
			this.mode = 'saveClient'
			this.phoneNumberMask = '(##) #####-####'
            this.client = {}
            this.date = ''
			this.isDisabled = true
			this.$refs.form.reset()
            this.$refs.listagem.resetClient()
			this.dialog = false
        },
		//metodo responsável por persistir os dados na base
		async saveClient(){
			let cpfExists = false
			
			if(this.mode === 'saveClient' || this.changeCpf){
				cpfExists = await this.cpfExists()
			}

			if(!cpfExists){
				const method = this.client.id ? 'put' : 'post'
				const id = this.client.id ? `/${this.client.id}` : ''
				this.client.telefone = this.client.telefone ? this.client.telefone.replace(/[^\d]/g, "") : ''
				this.client.cpf = this.client.cpf.replace(/[^\d]/g, "")
				this.client.dateFormatted = moment(this.client.dateFormatted, 'DD/MM/YYYY').format('YYYY-MM-DD')

				axios[method](id === '' ? `${baseApiUrl}/clients` : `${baseApiUrl}/clients${id}`, this.client)
					.then(() => {
						Swal.fire({
							position: 'center',
							icon: 'success',
							title: 'Cliente salvo com sucesso!',
							showConfirmButton: false,
							timer: 1500
						}).then(() => {
							this.resetClient()
						})
					})
					.catch(() => {
						Swal.fire({
							icon: 'error',
							title: 'Oops...',
							text: 'Contate o Administrador!',
						})
					})
			}else{
				Swal.fire({
					icon: 'error',
					title: 'Oops...',
					text: 'CPF já cadastrado!',
				})
			}
        },
		//metodo responsável por atualizar os dados na base
		updateDialog(event){
			this.mode = "updateClient"
			this.dialog = true
			this.oldCpf = event.cpf
			this.client = event
			//atualiza a mascara do campo telefone
			this.$nextTick(() => {
				this.$refs.telefone.reset()
				this.phoneNumberMask = '(##) #####-####'
			})
			this.date = moment(event.data_nascimento, 'DD/MM/YYYY').format('YYYY-MM-DD')
		},
		async cpfExists(){
			const url = `${baseApiUrl}/clients/${this.client.cpf.replace(/[^\d]/g, "")}`
			return await axios.get(url).then((res) => {
				return res.data.exists
			})
		}
    }
}
</script>

<style>
.v-data-table-header {
	background-color: #1E85DF !important
}
.theme--light.v-data-table>.v-data-table__wrapper>table>thead>tr>th {
	color: white;
}
.v-dialog > .v-card > .v-card__text {
    padding: 24px 20px;
}
</style>
