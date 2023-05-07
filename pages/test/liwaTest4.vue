<script setup lang="ts">
	/*********************************************************
	prog name: 測試SingleCol 元件, author: James Lin, date: 2022/12/05
	To Do: 測試 SingleCol 元件
	**********************************************************/
	import { ref, onMounted } from "vue"
	import { useFetch, createFetch } from "@vueuse/core"	
	import { IconPlusLg, IconTrash, IconX, IconAirplane } from '@iconify-prerendered/vue-bi'	
	import gemSysData from "../../static/gemSys4.json"

	const fieldCol = ['產地']
	const gridTitle = ref('產地資料設定')
	const phpurl = ref('021_D5edit.php')
	const liwaData = ref([])
	const itemVal = ref('')
	const action = ref('view')

	const loadData = async () => {
		let objItem = {
			'siteID': window.sessionStorage.getItem('liwaSiteID'),
			'action': action.value
		}
		let datastr = JSON.stringify(objItem)
	    const useMyFetch = createFetch({
	      baseUrl: window.sessionStorage.getItem('liwaAPIsvr'),
	      fetchOptions: {
	        mode: 'cors',
	        headers: new Headers({
	          'Content-Type': 'application/json; charset=utf-8'
	        }),
	        body: datastr
	      }
	    })
	    const { data } = await useMyFetch(phpurl.value).post().json() 
		liwaData.value = data.value.arrSQL
		liwaData.value.forEach((item) => {
			item.isShow = 0
			item.trCSS = ''
			item.boxCSS = 'top-12'
		})
	}

	const addD1 = () => {
		let nuNum = liwaData.value.length + 1
		let objItem = {
			'mainID':'',
			'itemNM':'',
			'isShow':1,
			'trCSS':'h-24',
			'boxCSS':'top-0'
		}
		liwaData.value.push(objItem)
		action.value = 'add'
		itemVal.value = ''
	}

	const saveData = async (idx) => {
		let keydata = {
			"action":action.value,
			"siteID":window.sessionStorage.getItem('liwaSiteID'),
			"mainID":liwaData.value[idx].mainID,
			"itemNM":itemVal.value
		}
		let datastr = JSON.stringify(keydata)
	    const useMyFetch = createFetch({
	      baseUrl: window.sessionStorage.getItem('liwaAPIsvr'),
	      fetchOptions: {
	        mode: 'cors',
	        headers: new Headers({
	          'Content-Type': 'application/json; charset=utf-8'
	        }),
	        body: datastr
	      }
	    })
	    const { data } = await useMyFetch(phpurl.value).post().json()
	    let msg = data.value.message
	    if (!msg) {
	    	if (action.value == 'add') {
	    		liwaData.value[idx].mainID = data.value.key
	    	}
			liwaData.value[idx].itemNM = itemVal.value
			liwaData.value[idx].isShow = 0
			liwaData.value[idx].trCSS = 'h-12'	    	
	    }
	    action.value = 'view'
	    itemVal.value = ''
	}

	const setMainID = (idx) => {
		// 先關閉所有的編輯盒
		liwaData.value.forEach((item) => {
			item.isShow = 0
			item.trCSS = ''
		})
		liwaData.value[idx].isShow = 1
		itemVal.value = liwaData.value[idx].itemNM
		liwaData.value[idx].trCSS = 'h-40'
		liwaData.value[idx].boxCSS = 'top-12'
		action.value='edit'
	}

	const closeBox = (idx) => {
		liwaData.value[idx].isShow = 0
		if (action.value == 'add') {
			liwaData.value.pop()
		}
		action.value = 'view'
		liwaData.value[idx].trCSS = ''
	}

	const itemDelete = async (idx) => {
		action.value = 'delete'
		let keydata = {
			"action":action.value,
			"siteID":window.sessionStorage.getItem('liwaSiteID'),
			"mainID":liwaData.value[idx].mainID
		}
		let datastr = JSON.stringify(keydata)
	    const useMyFetch = createFetch({
	      baseUrl: window.sessionStorage.getItem('liwaAPIsvr'),
	      fetchOptions: {
	        mode: 'cors',
	        headers: new Headers({
	          'Content-Type': 'application/json; charset=utf-8'
	        }),
	        body: datastr
	      }
	    })
	    const { data } = await useMyFetch(phpurl.value).post().json()
	    let msg = data.value.message
	    if (!msg) {
	    	liwaData.value.splice(idx, 1)
	    }
		action.value = 'view'
	}

	onMounted(() => {
		loadData()
	})

	definePageMeta({
	  title: 'liwaScore 測試',
	  layout: "default",
	})		

</script>

<template>
	<div class="w-full h-auto bg-gray-100">
		<div class="w-[900px] h-full mx-auto my-0 bg-white relative">
			<!-- 先設定 Title & btnAdd -->
			<div class="barPanel w-full h-12 rounded-3xl mt-2 mb-2 px-1 flex flex-row justify-between relative">
				<div class="w-full text-center">{{ gridTitle }}</div>
				<div class="top-icon Dadd absolute left-2 top-2 pl-[.125rem] pt-[.125rem]" @click="addD1()">
					<IconPlusLg class="w-7 h-7 text-slate-100 font-bold" />
				</div>				
			</div>
			<div v-if="liwaData.length > 0" class="w-full h-[calc(100vh_-_11rem)] overflow-x-hidden overflow-y-auto border-2 border-slate-400">
				<table class="w-full divide-x divide-y divide-gray-200 bg-white" :class="tableCSS">
					<thead>
						<th></th>
					</thead>
					<tbody>
						<tr class="relative odd:bg-white even:bg-slate-200"
							v-for="(item, index) in liwaData"
							:key="index"
							:data-id="item.mainID"
							:class="item.trCSS"
						>
							<td scope="col" class="w-full h-12 pt-3 pl-2 border-b-2 border-b-slate-300 flex flex-row">
								<div class="w-[calc(100%_-_3rem)] h-12" 
									@click="setMainID(index)">
									{{ item.itemNM }}
								</div>
								<div class="w-12 h-12 pt-[0.125rem] pr-[0.125rem] " @click="itemDelete(index)">
									<IconTrash class="w-7 h-7 text-red-300 font-bold" />
								</div>
							</td>
							<div v-if="item.isShow==1" class="absolute w-full h-auto left-0 bg-white z-50" 
							:class="item.boxCSS"
							>
								<div class="w-full bg-yellow-200">
									<div class="w-[calc(100%_-_2rem)] h-12 flex flex-row pt-2">
										<span class="pt-2 mx-2">{{ fieldCol[0] }}</span>
										<FormKit 
											type="text"
											label=""
											v-model="itemVal"
											outer-class="w-[calc(100%_-_8rem)]"
										/>
									</div>
									<div class="w-full h-16 flex flex-row">
										<div class="w-36 h-12 mt-2">
											<FormKit 
												type="submit"
												label="存檔"
												@click="saveData(index)"
											/>
										</div>
										<div class="w-8 h-12 pt-2 mt-2" @click="closeBox(index)">
											<IconX class="w-7 h-7 text-red-400 font-bold" 
											/>
										</div>
									</div>
								</div>
							</div>
						</tr>
					</tbody>
				</table>
			</div>
		</div>
	</div>

</template>