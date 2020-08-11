<template>
	<view class="content">
		<!-- <mix-tree :list="list" @treeItemClick="treeItemClick"></mix-tree> -->
		<ly-tree v-if="treeData.length > 0" :ready="ready" :props="props" :tree-data="treeData" node-key="id" @node-expand="handleNodeExpand" @node-click="handleNodeClick" />
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
		handleNodeClick(obj) {
			// #ifdef APP-PLUS
			console.log('handleNodeClick', JSON.stringify(obj));
			// #endif

			// #ifndef APP-PLUS
			console.log('handleNoiiiiideClick', obj);
			uni.navigateTo({
				url: '../listdesc/listdesc?itemDetail=' + encodeURIComponent(JSON.stringify(obj.data))
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
				for (let j = 0; j < data.length; j++) {
					let child_count = data[j];
					if (child_count.is_leaf === '否') {
						this.$set(child_count, 'children', []);
					}
					let count_no = await this.getTreeDataCount(child_count);

					// .then(count_no=>{
					this.$set(child_count, 'num', 0);
					child_count.num = 0;
					if (count_no) {
						this.$set(child_count, 'num', count_no.id);
					}
					// })
				}
				// data.forEach(child_count=>{
				// 	if(child_count.is_leaf === '否'){
				// 		this.$set(child_count,'children',[])
				// 	}
				// 	// this.getTreeDataCount(child_count).then(count_no=>{
				// 	// 	this.$set(child_count,'num',0)
				// 	// 	child_count.num = 0
				// 	// 	if(count_no){
				// 	// 		this.$set(child_count,'num',count_no.id)
				// 	// 	}
				// 	// })
				// })
				console.log('--------', data);
				e.data.children = data;
				// setTimeout(()=>{
				// 	let listData = []
				// 		data.forEach(item=>{
				// 			if(item.is_leaf == '否'){
				// 				item.children = []
				// 			}
				// 			e.child.push(item)
				// 		})
				// 		console.log('e.child',e)
				// },2000)
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
				
				// let count_no = await this.getTreeDataCount();
				// let totalNum = 0;
				
				// data.forEach(all=>{
				// 	 totalNum = 0;
				// 	for(let a = 0; a < count_no.length; a++){
				// 		totalNum += count_no.id;						
				// 		if(all.path_name === count_no[a].area_id){
				// 			this.$set(all,'num',count_no[a].id)
				// 		}
				// 	}
				// 	if(all.no === '610800000000'){
				// 		all.children = []
				// 		this.$set(all,'num',totalNum)
				// 		listData.push(all)						
				// 	} else if(all.is_leaf == '否'){
				// 		all.children = [];
				// 		listData[0].children.push(all);
				// 	}
				// })
								
				for (let i = 0; i < data.length; i++) {
					let nums = data[i];
					let count_no = await this.getTreeDataCount(nums)
					this.$set(nums, 'num', 0);
					nums.num = 0;
					if (count_no) {
						this.$set(nums, 'num', count_no.id);
					}
					console.log('count_num', count_no);
				}
				data.forEach(item => {
					if (item.no == '610800000000') {
						item.children = [];
						listData.push(item);
					} else {
						if (item.is_leaf == '否') {
							item.children = [];
						}
						listData[0].children.push(item);
					}
				});
				console.log('res.data', listData);
				this.ready = true;
				this.treeData = listData;
			
			}
		},
		async getTreeDataCount(area) {
			let urls = this.getServiceUrl('daq', 'srvdaq_tkry_people_select', 'select');
			let reqs = {
				serviceName: 'srvdaq_tkry_people_select',
				colNames: ['*'],
				condition: [{
					colName:"area_id",
					ruleType:"like",
					value:area.path_name
				}],
				group: [
					// {
					// 	colName: 'area_id',
					// 	type: 'by'
					// },
					{
						colName: 'id',
						type: 'count'
					}
				]
			};
			let res = await this.$http.post(urls, reqs);
			if (res.data.state === 'SUCCESS') {
				// let num = res.data.data;
				let num = res.data.data[0]
				return num;
			}
		},
		/** 组装地区数据*/
		assemblyData(parentNo, areaTreeData, itemData) {
			debugger;
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
