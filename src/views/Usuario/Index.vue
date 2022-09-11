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
								@click="resetUser"
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
											<v-form ref="form" id="cadUsuarios" @submit.prevent="submit">
												<input type="hidden" v-model="user.id">
												<v-row>
													<v-col cols="12" md="6" sm="12" xs="12">
															<v-text-field prepend-icon="mdi-account" label="Nome Completo" name="nome" type="text" v-model="user.nome" :rules="rules"></v-text-field>
													</v-col>
													<v-col cols="12" md="6" sm="12" xs="12">
															<v-text-field label="CPF" name="cpf" type="cpf" v-model="user.cpf" 
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
																v-model="user.dateFormatted"
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
															v-model="user.telefone" :rules="rulesTelefone"></v-text-field>
													</v-col>
												</v-row>
												<v-row>
													<v-btn @click="saveUser" :disabled="isDisabled" color="primary">Salvar</v-btn>
												</v-row>
											</v-form>
										</v-col>
									</v-row>
								</v-card-text>
								<v-card-actions class="justify-end">
								<v-btn
									text
									@click="resetUser"
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
				:perSelected="perSelected"
				:itemsPerPage="itemsPerPage"/>
				<Listagem ref="listagem" :search="search" :dialog="dialog" :user="user" @update-dialog="updateDialog($event)"/>
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
	nome: 'UsuarioIndex',
	components: { Search, Listagem },
	data(){
		return {
			mode: 'saveUser',
			dialog: false,
			panel: 1,
			showPassword: false,
			isDisabled: true,
			user: {},
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
			menu1: false,
			menu2: false,
			perSelected: {text: 10, value: 10},
			search: '',
			itemsPerPage: [
				{text: 5, value: 5},
				{text: 10, value: 10},
				{text: 15, value: 15},
				{text: 'All', value: -1}
			],
			phoneNumberMask: '',
			changeCpf: false
		}
	},
	watch: {
		menu (val) {
			val && setTimeout(() => (this.activePicker = 'YEAR'))
		},
		date () {
			this.user.dateFormatted = this.formatDate(this.date)
			if(this.user.cpf != "" && this.user.nome != "" && this.date != "" && this.user.telefone){
				this.isDisabled = false
			}else{
				this.isDisabled = true
			}
		},
		user: {
			handler(val){
				if(val.cpf && val.nome && val.dateFormatted && val.telefone){
					if(val.cpf != "" && val.nome != "" && val.dateFormatted != "" && val.telefone){
						this.isDisabled = false
					}else{
						this.isDisabled = true
					}
				}
			},
			deep: true
		},
		'user.cpf'(newVal, oldVal){
			if(newVal && oldVal){
				if(newVal.replace(/[^\d]/g, "") !== oldVal.replace(/[^\d]/g, "")){
					this.changeCpf = true
				}else{
					this.changeCpf = false
				}
			}
		},
    },
    methods: {
		saveDate (date) {
			this.$refs.menu.save(date)
		},
		formatDate (date) {
			if (!date) return null

			const [year, month, day] = date.split('-')
			return `${day}/${month}/${year}`
		},
		parseDate (date) {
			if (!date) return null

			const [month, day, year] = date.split('/')
			return `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`
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
		resetUser(){
			this.mode = 'saveUser'
			this.phoneNumberMask = '(##) #####-####'
            this.user = {}
            this.date = ''
			this.isDisabled = true
			this.$refs.form.reset()
            this.$refs.listagem.resetUser()
			this.dialog = false
        },
		async saveUser(){
			let cpfExists = false
			
			if(this.mode === 'saveUser' || this.changeCpf){
				cpfExists = await this.cpfExists()
			}

			if(!cpfExists){
				const method = this.user.id ? 'put' : 'post'
				const id = this.user.id ? `/${this.user.id}` : ''
				this.user.telefone = this.user.telefone.replace(/[^\d]/g, "")
				this.user.cpf = this.user.cpf.replace(/[^\d]/g, "")
				this.user.dateFormatted = moment(this.user.dateFormatted, 'DD/MM/YYYY').format('YYYY-MM-DD')

				axios[method](id === '' ? `${baseApiUrl}/users` : `${baseApiUrl}/users${id}`, this.user)
					.then(() => {
						Swal.fire({
							position: 'center',
							icon: 'success',
							title: 'Usuário salvo com sucesso!',
							showConfirmButton: false,
							timer: 1500
						}).then(() => {
							this.resetUser()
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
		updateDialog(event){
			this.mode = "updateUser"
			this.dialog = true
			this.user = event
			this.$nextTick(() => {
				this.$refs.telefone.reset()
				this.phoneNumberMask = '(##) #####-####'
			})
			this.date = moment(event.data_nascimento, 'DD/MM/YYYY').format('YYYY-MM-DD')
		},
		async cpfExists(){
			const url = `${baseApiUrl}/users/${this.user.cpf.replace(/[^\d]/g, "")}`
			return await axios.get(url).then((res) => {
				return res.data.exists
			})
		}
    },
	computed: {
      computedDateFormatted () {
        return this.formatDate(this.date)
      },
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
