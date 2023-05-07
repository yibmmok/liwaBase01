<script setup lang="ts">
	/*********************************************************
	prog name: 測試liwaTbl元件, author: James Lin, date: 2023/01/23
	To Do: 1. 顯示table; 2.排序; 3.搜尋(搜尋條件設到params參數內); 4.
	**********************************************************/
	import { ref, onMounted } from "vue"
	import liwaGrid from "../../components/liwaGrid.vue"

	const APIsvr = ref('')
	const mainID = ref('10010142')
	const page = ref(1)
	const pageSize = ref(10)
	const filterArray = ref([])

	const testProc = () => {

	}

	const setMainID = (sID) => {
		window.location.href = '/002/'+sID
	}

	const setActvPage = (iPage) => {
		page.value = iPage
	}

	const setMsg = (sMsg) => {
		alert(sMsg)
	}

	onMounted(() => {
		useHead({title:'測試程式2'})
		APIsvr.value = window.sessionStorage.getItem('liwaAPIsvr')
	})

	definePageMeta({
	  layout: "default",
	  colorMode: "light"
	})		

</script>

<template>
	<div class="w-full">
		<div class="w-24 py-2 text-center border-2 border-slate-800 rounded-md" @click="testProc()">測試</div>		
	</div>
	<div class="w-full h-[calc(100vh_-_130px)] relative">

		<div class="w-full mx-auto">
			<liwaGrid
				:tblTitle="'待辦事項明細表'"
				:progID="'002_M'"
				:viewUrl="'002_havelist.php'"
				:params="''"
				:page="page"
				:pageSize="pageSize"
				:filterArray="filterArray"
				@sendMsg="sendMsg"
				@setActvPage="setActvPage"
				@setMainID="setMainID"
			/>
		</div>
	</div>
</template>