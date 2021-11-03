<template>
<div class="container">
	<div class="filter-input">
		<span>Фильтрация по : </span>
		<input v-model="filter" type="text" id = "fllter">
	</div>
	<table class="table">
		<thead>
			<tr>
				<th @click="Sort()">id</th>
				<th @click="Sort()">type</th>
				<th @click="Sort()">name</th>
			</tr>
		</thead>
		<tbody>
			<tr v-for="data in TableData" :key="data.id">
				<td>{{data.id}}</td>
				<td>{{data.type}}</td>
				<td>{{data.attributes.canonicalTitle}}</td>
			</tr>
		</tbody>
		<div class="navigation-page">
			<a @click = "toPage(prevPage)" class = "navigation-link">Предыдущая страница</a>
			<ul class = "navigation-list">
				<li :class="count_page == page ? 'active' : ''" @click = "toPage(page)" class = "navigation-item" v-for="page in lastPage" :key="page.id">
					{{page}}
				</li>
			</ul>
			<a @click = "toPage(nextPage)" class = "navigation-link">Следущая страница</a>
		</div>
	</table>
</div>
</template>

<script>
// @ is an alias to /src
export default {
	name: 'Home',
	data () {
		return {
		TableData: [],
		filterArray : [],
		lastPage: 1,
		nextPage : '',
		prevPage : '',
		count_page : 1,
		filter : '',
		}
	},
	async mounted () {
		let responceData = await this.GetData('https://kitsu.io/api/edge/anime?page[limit]=20page[offset]=0');
		this.lastPage = Math.floor(responceData.meta.count / 20);
		this.nextPage = responceData.links.next;
		this.TableData = responceData.data;
		this.filterArray = this.TableData;
	},
	methods : {
		async GetData (url) {
			let responce = await fetch(url, {
				"method": "GET",
				"headers": {
					"x-rapidapi-host": "jikan1.p.rapidapi.com",
					"x-rapidapi-key": "a036482400msh26ea87843768135p14c29djsn91a021013a35"
				}
			})
			responce = await responce.json();
			return responce
		},
		async toPage (page) {	
			if(isNaN(page)) {
				let parseCount = (page.split('=')[2] / 20) + 1;
				this.count_page = parseCount
			} else {
				this.count_page = page;
				page = (page - 1) * 20;
			}
			let responceData = await this.GetData(`https://kitsu.io/api/edge/anime?page%5Blimit%5D=20&page%5Boffset%5D=${page}`);
			this.nextPage = responceData.links.next;
			this.prevPage = responceData.links.prev;
			this.TableData = responceData.data;
			this.filterArray = this.TableData;
		},
		Sort() {
			this.TableData = this.TableData.sort((a, b) => a > b ? 1 : -1);
		},
		checkExist: function(event){
			let value = event.target.value;
			console.log(value);
		}
	},
	watch: {
		filter : function (value) {
			this.TableData = [];
			this.filterArray.filter((data) => {
				let name = data.attributes.canonicalTitle.includes(value);
				name ? this.TableData.push(data) : '';
			})
		}
	}

}
</script>
<style scoped>
.table {
	width: 100%;
	border: none;
	margin-bottom: 20px;
}
.table thead th {
	font-weight: bold;
	text-align: left;
	border: none;
	padding: 10px 15px;
	background: #d8d8d8;
	font-size: 14px;
}
.table thead tr th:first-child {
	border-radius: 8px 0 0 8px;
}
.table thead tr th:last-child {
	border-radius: 0 8px 8px 0;
}
.table tbody td {
	text-align: left;
	border: none;
	padding: 10px 15px;
	font-size: 14px;
	vertical-align: top;
}
.table tbody tr:nth-child(even){
	background: #f3f3f3;
}
.table tbody tr td:first-child {
	border-radius: 8px 0 0 8px;
}
.table tbody tr td:last-child {
	border-radius: 0 8px 8px 0;
}
.navigation-page {
	display: flex;
	flex-direction: row;
}
.navigation-link {
	cursor: pointer;
}
.navigation-link:first-child {
	margin-right: 1rem;
}
.navigation-count {
	margin-right: 1rem;
}
.navigation-list {
	display: flex;
	flex-direction: row;
	list-style: none;
	width: 500px;
	overflow: auto;
}
.navigation-item {
	padding: 0.3rem;
	cursor: pointer;
	background: transparent;
	transition: all ease 0.3s;
}
.navigation-item:hover {
	background: gray;
}
.navigation-item.active {
	background: green;
}
</style>
