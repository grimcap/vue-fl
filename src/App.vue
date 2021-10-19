<template>
	<div class="dataCont">
		<div class="filterCont">
			<ui-textfield outlined
			>Фильтр по тексту</ui-textfield>
			<ui-form-field>
				<ui-checkbox input-id="filteredByCost"></ui-checkbox>
				<label for="checkbox">Сначала дешевле</label>
			</ui-form-field>
		</div>

		<ui-table :data="itms" :thead="thead" :tbody="tbody" @click="tableClick($event.target)" class="projTable">
			<template #skills="{data}">
				<span v-for="skill in data.skills" :key="skill.id" class="proj_cat">{{skill.name}}</span>
			</template>
		</ui-table>
	</div>

	<ui-dialog v-model="popUpOpen" class="popUp">
		<ui-dialog-title>{{this.curItm.name}}</ui-dialog-title>
		<ui-dialog-content>
			<small class="published">{{formatDate(this.curItm.published_at)}}</small>
			<p v-html="this.curItm.description_html"></p>
		</ui-dialog-content>
		<ui-dialog-actions>
			<ui-button @click="popUpOpen = false">OK</ui-button>
		</ui-dialog-actions>
	</ui-dialog>
</template>

<script>
// Vue.http.headers.common['Access-Control-Allow-Origin'] = '*'

export default {
	name: 'SomeTable',

	data: () => ({
		data_json: 'http://localhost:3000/projects', //'https://jsonplaceholder.typicode.com/posts',
		itms: [],
		errors: [],
		curItm: [],

		popUpOpen: false,

		thead: [
			'Название',
			'Категории',
			'Бюджет',
		],
		tbody: [
			{
				field: 'id',
				class: 'row_id'
			},
			'name',
			{
				slot: 'skills',
			},
			{
				field: 'budget',
				fn: (itms) => {
					return itms.budget.amount +' '+ itms.budget.currency;
				}
			}
		]
	}),

	created() {
		this.getItems();
	},

	methods: {
		getItems() {
			fetch(this.data_json, {
				headers: {
					'Content-type': 'application/json',
				},
			}).then(res=>res.json()).then((response) => {
				console.log(response)
				this.itms = response;
			}).catch( (e) => {
				console.log(e)
				// this.errors.push(e)
			});
		},

		tableClick(row) {
			if(!row.hasAttribute('role')){
				this.curItm_id = row.closest('tr').querySelector('.row_id').innerText*1;
				this.curItm = (this.itms.filter((el) => el.id === this.curItm_id))[0];
				this.popUpOpen = true;
			}
		},

		formatDate(date) {
			this.d = new Date(date);
			return this.d.getFullYear() +'-'+ this.d.getMonth() +'-'+ this.d.getDate();
			// d => d.toLocaleString('ru-RU').replace(',', '').slice(0, -3)
		}
	}
}
</script>

<style lang="less">
	.dataCont {
		display: flex;
		/* justify-content: center; */
		flex-direction: column;
		align-items: center;
		margin-top: 10%;
	}

	.filterCont {
		display: flex;
		justify-content: space-between;
		width: 100%;
		max-width: 520px;
		margin-bottom: 30px;
	}

	.projTable {
		cursor: pointer;

		.mdc-data-table__header-cell {
			background-color: #6200ee11;
		}

		.row_id {display: none}

		.mdc-data-table__row {
			transition: .3s ease-out;
		}
	}

	.popUp .published {
		display: block;
		text-align: right;
	}
	
</style>
