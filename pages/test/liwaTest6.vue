<script setup lang="ts">
	/*********************************************************
	prog name: 測試SingleCol 元件, author: James Lin, date: 2022/12/05
	To Do: 測試 SingleCol 元件
	**********************************************************/
	import { ref, onMounted } from "vue"
	import { useFetch, createFetch } from "@vueuse/core"	
	import liwa2ColList from "../../components/liwa2ColList"
	import { IconPlusLg, IconTrash, IconX, IconAirplane } from '@iconify-prerendered/vue-bi'

	const siteID = ref('')
	const liwaD8 = ref([])
	const bShow = ref(false)

	const loadD8 = async () => {
		let url= window.sessionStorage.getItem('liwaAPIsvr') + "/021_haveD8.php?siteID="+siteID.value
		const dataD8 = await useFetch(url, {method: 'GET'}, {refetch: true}).get().json()	
		liwaD8.value = dataD8.data.value.arrSQL		
	}

	const renewD8 = (arrD8) => {
		liwaD8.value = []
		arrD8.forEach((item) => {
			let objItem = {
				'item1': item.item1,
				'item2': item.item2,
				'item3': item.item3,
			}
			liwaD8.value.push(objItem)
		})
		console.log('liwaD8 =', liwaD8.value)
	}

	onMounted(() => {
		siteID.value = window.sessionStorage.getItem('liwaSiteID')
		// loadD8()

	})

	definePageMeta({
	  title: 'liwa2ColList 測試',
	  layout: "default",
	})		

</script>

<template>
	<div class="w-full h-auto bg-gray-100">
		<div class="w-[800px] h-28 mx-auto ">
			<liwa3ColList
				:headCol="['收費', '金額', '發票號碼']"
				:gridTitle="'收費明細表'"
				:phpurl="'021_D8edit.php'"
				:params="['10000013']"
				@setList="renewD8"
			/>				
		</div>		
	</div>
</template>

<style scope>
	@import "../../assets/css/multistep-form.css";
</style>