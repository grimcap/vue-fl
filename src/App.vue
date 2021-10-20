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
			<template #name="{data}" :class="aaa">
				{{data.attributes.name}}
			</template>
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
		<ui-dialog-title>{{this.itms[curItm] && this.itms[curItm].attributes.name}}</ui-dialog-title>
		<!-- <ui-dialog-content>
			<small class="published">{{formatDate(this.curItm.attributes.published_at)}}</small>
			<div v-html="this.curItm.attributes.description_html" class="popUp_content"></div>
		</ui-dialog-content> -->
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
		curItm: 0,

		popUp_opened: false,

		thead: [
			'id',
			'Название',
			'Категории',
			// 'Бюджет',
			{
				value: 'Бюджет',
				// sort: 'none',
			}
		],
		tbody: [
			{
				field: 'id',
				class: 'row_id'
			},
			{
				// slot: 'name',
				field: 'name',
				class: (itms) => {return 'id-'+ itms.id},
				fn: (itms) => {
					return itms.attributes.name
				}
			},
			{
				slot: 'skills',
			},
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
					'Authorization': 'Bearer fa362b273b45ef958a3fb34aad0b1eb25abd6e1e'
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
			if(!row.hasAttribute('role')){
				console.log(row.closest('tr').classList)
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
		cursor: pointer;

		.mdc-data-table__header-cell {
			background-color: #6200ee11;
		}

		/* .row_id {display: none} */

		.mdc-data-table__row {
			transition: .3s ease-out;
		}

		.proj_cat + .proj_cat:before {
			content: ', ';
		}
	}

	.popUp .published {
		display: block;
		text-align: right;
	}
	
</style>
