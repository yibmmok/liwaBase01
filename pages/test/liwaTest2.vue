<script setup lang="ts">
	/*********************************************************
	prog name: 測試liwaTbl元件, author: James Lin, date: 2023/01/23
	To Do: 1. 顯示table; 2.排序; 3.搜尋(搜尋條件設到params參數內); 4.
	**********************************************************/
	import { ref, onMounted } from "vue"
	import liwaGrid from "../../components/liwaGrid.vue"
	import liwaMsg from "../../components/liwaMsg.vue"

	const APIsvr = ref('')
	const mainID = ref('10010142')
	const progID = ref('021')
	const page = ref(1)
	const pageSize = ref(10)
	const filterArray = ref([])
	const isFilter = ref(false)
	const filters = ref({
		"prodID":"",
		"prodNM":"",
		"expDate1":"",
		"expDate2":"",
		"sellerID":"",
		"status":1
	})
	const isGrid = ref(true)
	const optStatus = ref([{"label":"下架", "value":"0"}, {"label":"上架", "value":"1"}, {"label":"斡旋中", "value":"2"}, {"label":"成交", "value":"9"}])	
	// liwaMsg 初始值
	const isMsg = ref(false)
	const objMsg = reactive({
		title: '',
		body: '',
		modalType: 1
	})

	// 設定 liwaMsg starts
	const showMsg = (sTitle, sBody, iType = 1) => {
		objMsg.title = sTitle
		objMsg.body = sBody
		objMsg.modalType = iType
		isMsg.value = true
	}

	const hideMsg = () => {
		isMsg.value = false
	}

	const confirmOK = () => {
		isMsg.value = false
	}	
	// 設定 liwaMsg ends

	// 設定 filters 內容 starts
	const toggleFilter = () => {
		// 顯示 filters 對話盒
		isFilter.value = !isFilter.value
	}	

	const goSearch = () => {
		// 顯示結果並關閉對話盒
		isFilter.value = false
	}
	// 設定 filters 內容 ends	

	const testProc = () => {
		toggleFilter()
	}

	const setMainID = (sID) => {
		if (sID == 'filter') {
			isFilter.value = true
		} else {
			window.location.href = `/${progID.value}/`+sID
		}
	}

	const setMsg = (sMsg) => {
		showMsg('程式錯誤', sMsg, 1)
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
	<div v-if="isGrid" class="w-full h-[calc(100vh_-_130px)] relative">
		<div class="w-full mx-auto">
			<liwaGrid
				:tblTitle="'物件列表'"
				:progID="'021_M'"
				:viewUrl="'021_havelist.php'"
				:params="''"
				:page="page"
				:pageSize="pageSize"
				:filterArray="filterArray"
				@sendMsg="sendMsg"
				@setMainID="setMainID"
			/>
		</div>
	</div>
	<teleport to="body">
		<liwaMsg 
		  	v-if="isMsg"
		  	:msgTitle="objMsg.title"
		  	:msgBody="objMsg.body"
		  	:modalType="objMsg.modalType"
		  	@hideMsg="hideMsg"
		  	@confirmOK="confirmOK"
		/>  	
	</teleport>	
	<teleport to="body">
		<div v-if="isFilter" class="absolute top-[170px] lg:top-[110px] z-10 inset-x-0 w-screen h-[calc(100vh_-_79px)] bg-transparent overflow-hidden items-center justify-center">
		    <div class="flex justify-center w-full h-screen bg-transparent items-start antialiased">
		      	<div class="h-full lg:h-[calc(70vh_-_2rem)] flex flex-col mt-4 w-11/12 sm:w-5/6 lg:w-1/2 max-w-lg mx-auto rounded-lg border border-gray-300 shadow-xl">
		        	<div class="relative flex flex-row justify-between px-6 py-2 bg-white border-b border-gray-200 rounded-tl-lg rounded-tr-lg text-center ">
		        		<div class="w-5/7 h-8 text-2xl text-center">物件查詢</div>
		        		<div class="w-2/7 h-8 flex flex-row justify-between">
				            <div class="w-8 h-8 top-2 right-2 bg-white cursor-pointer" @click.prevent="toggleFilter()">
				            	<IconX class="w-7 h-7 text-red-400 font-bold" />
				            </div>          			
		        		</div>
		        	</div>
		        	<div class="w-full h-full bg-slate-100">
		        		<FormKit 
		        			form-class="mt-4 ml-4 px-4 py-2 bg-yellow-200 rounded-2xl w-11/12"
		        			type="form"
		        			v-model="filters"      			
		        			:form-class="submitted? 'hidden': 'block'"
		        			submit-label="查詢"
		        			@submit="goSearch()"
		        		>
					        <FormKit
					          name="prodID"
					          label="物件編號"
					          type="text"
					          placeholder="輸入物件編號"
					          help="請輸入物件編號"
					        />
					        <FormKit
					          name="prodNM"
					          label="物件名稱"
					          type="text"
					          placeholder="輸入物件名稱"
					          help="可輸入部份物件名稱關鍵字"
					        />
					        <div class="w-[95%] flex flex-col lg:flex-row justify-between">
						        <FormKit
						          name="expDate1"
						          label="刊登到期日(起始日期)"
						          type="date"
						        />
						        <FormKit
						          name="expDate2"
						          label="刊登到期日(結束日期)"
						          type="date"
						        />					        	
					        </div>
					        <FormKit
					          name="sellerID"
					          label="賣家ID"
					          type="text"
					          placeholder="輸入賣家ID"
					          help="請輸入賣家ID"
					        />
					        <FormKit
					          name="status"
					          label="物件狀態"
					          type="select"
					          help="請選擇物件狀態"
					          :options="optStatus"
					        />
		        		</FormKit>
		        	</div>
		        </div>
		    </div>
		</div>	
	</teleport>
</template>