<template>
	<view class="content">
		<!-- <mix-tree :list="list" @treeItemClick="treeItemClick"></mix-tree> -->
		<ly-tree v-if="treeData.length > 0" :ready="ready" :props="props" :tree-data="treeData" node-key="id" @item-detai='itemDetail' @node-expand="handleNodeExpand" @node-click="handleNodeClick" />
		<!-- <ly-tree :tree-data="treeData" node-key="id" :props="props" @node-expand="handleNodeExpand" @node-click="handleNodeClick" /> -->
		<!-- <pg-tree :list="list" :selected="selected" :unfoldPath="unfoldPath" :keepLevel="false" :isAllfold="2" :enableChoose="false" :finalChoose="false" :enableEdit="true" @itemClick="itemClick" @itemEdit="itemEdit" @switchFold="switchFold"></pg-tree> -->
	</view>
</template>

<script>
// import pgTree from '@/components/pg-tree/pg-tree';
// import mixTree from '@/components/mix-tree/mix-tree';
import LyTree from '@/components/ly-tree/ly-tree.vue';
export default {
	components: {
		// pgTree,
		// mixTree
		LyTree
	},
	data() {
		return {
			selected: [],
			ready: false,
			props: function() {
				return {
					label(data, node) {
						return node.data.name;
					},
					isLeaf(data, node) {
						let is_leafs = false;
						if (node.data.is_leaf === '是') {
							is_leafs = true;
						} else {
							is_leafs = false;
						}
						return is_leafs;
					}
				};
			},
			list: [],
			treeData: []
		};
	},
	onLoad() {
		this.getTreeRegionData();
	},
	methods: {
		loadNode(node, resolve) {},
		itemDetail(data){
			console.log("====》",data) 
			uni.navigateTo({
				url: '../listdesc/listdesc?itemDetail=' + encodeURIComponent(JSON.stringify(data.item))
			});
		},
		handleNodeClick(obj) {
			// #ifdef APP-PLUS
			console.log('handleNodeClick', JSON.stringify(obj));
			// #endif

			// #ifndef APP-PLUS
			console.log('handleNoiiiiideClick', obj);
			uni.navigateTo({
				url: '../listdesc/listdesc?itemDetail=' + encodeURIComponent(JSON.stringify(obj))
			});
			// #endif
		},

		handleNodeExpand(obj) {
			// #ifdef APP-PLUS
			console.log('handleNodeExpand', JSON.stringify(obj));
			// #endif

			// #ifndef APP-PLUS
			console.log('handleNodeExpand——obh', obj);
			if (obj.data.no != '610800000000') {
				this.getCilckChildData(obj);
			}
			// #endif
		},
		async getCilckChildData(e) {
			let urls = this.getServiceUrl('config', 'srvconfig_area_adj_select', 'select');
			let reqs = {
				serviceName: 'srvconfig_area_adj_select',
				colNames: ['*'],
				condition: [
					{
						colName: 'parent_no',
						ruleType: 'eq',
						value: e.data.no
					}
				]
			};
			let res = await this.$http.post(urls, reqs);
			if (res.data.state === 'SUCCESS') {
				let data = res.data.data;
				
				let poverty = await this.getTreeDataCount();
				
				let totalNum = 0;
				
				data.forEach(all=>{
					 let itemArea = 0;
					 totalNum = 0;
					 if(all.is_leaf === '否'){
						 this.$set(all, 'children', []);
					 }
					for(let a = 0; a < poverty.length; a++){		
						if(poverty[a].area_id.indexOf(all.path_name) !== -1){
							itemArea++
						}
					}
					this.$set(all, 'num', itemArea);						
				})
				
				console.log('--------', data);
				e.data.children = data;
			}
		},
		async getTreeRegionData() {
			let urls = this.getServiceUrl('config', 'srvconfig_area_adj_select', 'select');
			let reqs = {
				serviceName: 'srvconfig_area_adj_select',
				colNames: ['*'],
				// condition:[{
				// 	colName: "path_name",
				// 	ruleType: "like",
				// 	value: "/陕西省/榆林市/"
				// }]
				relation_condition: {
					relation: 'OR',
					data: [
						{
							colName: 'parent_no',
							ruleType: 'eq',
							value: '610800000000'
						},
						{
							colName: 'no',
							ruleType: 'eq',
							value: '610800000000'
						}
					]
				}
			};
			let res = await this.$http.post(urls, reqs);
			if (res.data.state === 'SUCCESS') {
				let data = res.data.data;
				let listData = [];
				
				let poverty = await this.getTreeDataCount();
				console.log("特困人员---",poverty)
				
				let totalNum = 0;
				
				data.forEach(all=>{
					 let itemArea = 0;
					 totalNum = 0;
					for(let a = 0; a < poverty.length; a++){
						// totalNum += count_no.id;						
						if(poverty[a].area_id.indexOf(all.path_name) !== -1){
							itemArea++
						}
					}
					this.$set(all, 'num', itemArea);
					if(all.no === '610800000000'){
						all.children = []
						this.$set(all,'num',poverty.length)
						listData.push(all)						
					} else if(all.is_leaf == '否'){
						all.children = [];
						listData[0].children.push(all);
					}
				})
				console.log('res.data', listData);
				this.ready = true;
				this.treeData = listData;
			
			}
		},
		async getTreeDataCount() { 
			let urls = this.getServiceUrl('daq', 'srvdaq_tkry_people_select', 'select');
			let reqs = {
				serviceName: 'srvdaq_tkry_people_select',
				colNames: ['*'],
				condition: [
				// 	{
				// 	colName:"area_id",
				// 	ruleType:"like",
				// 	value:area.path_name
				// },
				],
				// group: [
				// 	// {
				// 	// 	colName: 'area_id',
				// 	// 	type: 'by'
				// 	// },
				// 	{
				// 		colName: 'id',
				// 		type: 'count'
				// 	}
				// ]
			};
			let res = await this.$http.post(urls, reqs);
			if (res.data.state === 'SUCCESS') {
				let num = res.data.data;
				// let num = res.data.data[0]
				return num;
			}
		},
		/** 组装地区数据*/
		assemblyData(parentNo, areaTreeData, itemData) {
			let testData = [];
			testData.push(itemData);
			this.assemblyChildData(itemData, areaTreeData, parentNo);
			return testData;
		},
		assemblyChildData(itemData, arr, parentNo) {
			itemData.children = itemData.children ? itemData.children : [];
			arr.forEach(items => {
				if (items.parent_no === parentNo) {
					itemData.children.push(items);
					this.assemblyChildData(items, arr, items.no);
				}
			});
			return itemData.children;
		},
		itemEdit(ev) {
			console.log(ev);
			if (ev.isEdit) {
				uni.showToast({
					title: '你在编辑:' + ev.item.name,
					duration: 2000
				});
			}
			if (ev.isDel) {
				uni.showModal({
					content: '您确定要删除' + ev.item.name + '?',
					showCancel: true,
					complete: e => {}
				});
			}
		},
		switchFold(e) {
			console.log('当前折叠路径为： ', e);
		}
	}
};
</script>

<style>
page {
	background-color: #f0f0f0;
}
</style>
