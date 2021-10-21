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
				<span v-for="skill in data.attributes.skills" :key="skill.id" class="proj_cat">{{skill.name}}</span>
			</template>
		</ui-table>
	</div>


	<!-- <div class="dataTable" v-for="itm in itms" :key="itm.id">
		<div class="col_name">{{itm.attributes.name}}</div>
		<div class="col_skills">
			<span class="skill" v-for="skill in itm.attributes.skills" :key="skill.id">{{skill.name}}</span>
		</div>
		<div class="col_budget">{{itm.attributes.budget && (itm.attributes.budget.amount +' '+itm.attributes.budget.currency)}}</div>
	</div> -->


	<ui-dialog v-model="popUp_opened" class="popUp">
		<ui-dialog-title>{{this.curItm.attributes && this.curItm.attributes.name}}</ui-dialog-title>
		<ui-dialog-content>
			<small class="published">{{formatDate(this.curItm.attributes && this.curItm.attributes.published_at)}}</small>
			<p v-html="this.curItm.attributes && this.curItm.attributes.description_html" class="popUp_content"></p>
			<a class="link" :href="this.curItm.links && this.curItm.links.self.web" target="_blank">Смотреть на Freelancehunt</a>

			<!-- <a class="link" href="#">Сылка</a> -->
		</ui-dialog-content>
		<ui-dialog-actions>
			<ui-button @click="popUp_opened = false">OK</ui-button>
		</ui-dialog-actions>
	</ui-dialog>
</template>

<script>
// Vue.http.headers.common['Access-Control-Allow-Origin'] = '*'

export default {
	name: 'SomeTable',

	data: () => ({
		data_url: 'https://api.freelancehunt.com/v2/projects',
		// data_url: 'http://localhost:3000/projects', 
		data_itms: [],
		itms: [],
		errors: [],
		curItm: [],

		popUp_opened: false,

		thead: [
			'id',
			{
				value: 'Название',
				class: 'col_name'
			},
			'Категории',
			{
				value: 'Бюджет',
				// sort: 'asc',
				// columnId: 'budget',
			},
			// 'Бюджет',
		],
		tbody: [
			{
				field: 'id',
				class: 'row_id'
			},
			{
				field: 'name',
				class: (itms) => {return 'col_name id-'+ itms.id},
				fn: (itms) => {
					return itms.attributes.name
				}
			},
			{
				slot: 'skills',
				class: 'col_skills',
			},
			// {
			// 	field: 'budget',
			// 	fn: (itm) => {
			// 		let cost = itm.attributes.budget && itm.attributes.budget.amount;
			// 		return cost;
			// 	}
			// },
			{
				field: 'budget',
				fn: (itm) => {
					return itm.attributes.budget && (itm.attributes.budget.amount +' '+itm.attributes.budget.currency);
				}
			}
		]
	}),

	created() {
		this.getItems();
	},

	methods: {
		getItems() {
			fetch(this.data_url, {
				headers: {
					// 'Content-type': 'application/json',
					'Authorization': 'Bearer '
				},
			}).then(res=>res.json()).then((response) => {
				console.log(response)
				this.data_itms = response.data;
				this.itms = this.data_itms;
			}).catch( (e) => {
				console.log(e)
				// this.errors.push(e)
			});
		},

		tableClick(row) {
			if(row.closest('tr').className != 'mdc-data-table__header-row'){
				this.curItm_id = row.closest('tr').querySelector('.row_id').innerText*1;
				this.curItm = (this.itms.filter((el) => el.id === this.curItm_id))[0];
				this.popUp_opened = true;
			}
		},

		formatDate(date) {
			this.d = new Date(date);
			return this.d.getFullYear() +'-'+ this.d.getMonth() +'-'+ this.d.getDate();
		}
	}
}
</script>

<style lang="less">
	#app {text-align: center}

	.dataCont {
		display: inline-flex;
		flex-direction: column;
		align-items: center;
		margin-top: 10%;
		max-width: 80%;
		font-size: 14px;
	}

	.filterCont {
		display: flex;
		justify-content: space-between;
		width: 100%;
		margin-bottom: 30px;
	}

	.dataTable {
		display: flex;
		/* grid-template */
		max-width: 520px;
		margin: 0 auto;
		border: 1px solid #ccc;

		div {
			width: 100px;
			padding: 16px;
		}
	}

	.projTable {
		max-width: 100%;
		cursor: pointer;

		.mdc-data-table__header-cell:first-child, .mdc-data-table__cell:first-child {display: none}

		.mdc-data-table__header-cell {
			background-color: #6200ee11;
		}

		.col_name {max-width: 300px}
		.col_skills {max-width: 300px}

		/* .row_id {display: none} */

		.mdc-data-table__row {
			transition: .3s ease-out;
		}

		.proj_cat + .proj_cat:before {
			content: ', ';
		}
	}

	.popUp {
		padding: 10px 15px;
		text-align: left;

		h2 {line-height: 1.3}

		.mdc-dialog__content {
			max-height: 50vh;
			word-wrap: break-word;
		}

		.published {
			display: block;
			margin-left: 10px;
			margin-bottom: 10px;
		}

		p {margin: 0 0 5px}
		p br {display: none}

		.link {
			position: absolute;
			bottom: 15px;
			left: 25px;
			z-index: 10;
			border: 0;
			outline: 0;
			text-decoration: none;
			border-bottom: 1px solid #0000;

			&:hover {
				border-bottom: 1px solid #6200ee;
				transition: .3s;
			}
		}
	}
	
</style>
