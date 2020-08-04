<template>
	<view class="bg-white form-box" style="width:100%;" v-if="allField.length > 0">
		<view v-for="(item, index) in allField" :key="index" class="form-box-item">
			<formItem
				:procData="procData"
				:field="item"
				:pageFormType="BxformType"
				:showTextarea="showTextarea"
				:fieldsModel="fieldModel"
				:service="service"
				:detailFiledData="detailFiledData"
				ref="fitem"
				@on-form-item="onItemButtons($event)"
				@on-value-change="onValChange($event)"
				@on-value-blur="onValBlur($event)"
				@get-cascader-val="getCascaderVal"
				@picker-change="pickerchange"
			></formItem>
		</view>
	</view>
</template>

<script>
import formItem from '@/components/bx-form/bx-form-item.vue';
import evaluatorTo from '@/common/evaluator.js';
export default {
	name: 'bx-form',
	components: { formItem },
	props: {
		fields: {
			type: Array,
			default() {
				return [];
			}
		},
		procData: {
			type: Object,
			default() {
				return {};
			}
		},
		defaultCondition: {
			type: Array,
			default() {
				return [];
			}
		},
		pageType: {
			type: String,
			default() {
				return '';
			}
		},
		BxformType: {
			type: String,
			default() {
				return 'form';
			}
		},
		service: {
			type: String,
			default() {
				return '';
			}
		},
		showTextarea: {
			type: Boolean,
			default: true
		},
		moreConfig: {
			type: Object,
			default() {
				return {};
			}
		},
		detailFiledData: {
			type: Object,
			default() {
				return {};
			}
		}
	},
	computed: {
		fieldsData: function() {
			return this.fields;
		}
	},
	data() {
		return {
			allField: this.fields,
			fieldModel: {},
			fieldData: {},
			oldField: [],
			oldFieldModel: {},
			specialCol: [],
			more_config: {
				service_call_cfg: [
				]
			}
		};
	},
	created() {
		if (this.fields.length > 0) {
			this.oldField = this.deepClone(this.fields);
			this.oldField.forEach((item, index) => {
				this.oldFieldModel[item.column] = item.value;
			});
		}
		this.getAllField();
	},
	mounted() {
		if (this.fields.length > 0) {
			this.oldField = this.deepClone(this.fields);
			this.oldField.forEach((item, index) => {
				this.oldFieldModel[item.column] = item.value;
			});
		}
	},
	methods: {
		pickerchange(oriData) {
			console.log('oriData------', oriData, this.allField);
			let filed = this.allField;
			filed.forEach(item => {
				if (item.column === 'dybm') {
					// item.poupValue = oriData.lybm
					this.$set(item, 'poupValue', oriData.lybm);
				} else if (item.column === 'fwbm') {
					this.$set(item, 'louValue', oriData.lybm);
					this.$set(item, 'dyValue', oriData.dybm);
				}
			});
			console.log(this.allField);
		},
		onItemButtons(e) {
			this.$emit('on-form-item', e);
			return e;
		},
		getAllField() {
			let self = this;
			console.log('getAllField', this.fields);
			console.log('111111111111111111111111', this.allField);
			if (this.fields.length > 0) {
				let fields = this.deepClone(this.fields);
				this.oldField = this.deepClone(this.fields);
				this.oldField.forEach((item, index) => {
					this.oldFieldModel[item.column] = item.value;
				});
				this.allField = fields.map((itemData, index) => {
					this.fieldModel[itemData.column] = itemData.value;
					this.$emit('changeFieldModel',this.fieldModel)
					let item = this.fieldModel;
					if (itemData.hasOwnProperty('isShowExp') && item.hasOwnProperty(itemData.column)) {
						itemData['showExp'] = this.evalInTo(itemData, item);
						// itemData['showExp'] = this.colItemShowExps(itemData, item) ;
						itemData['display'] = itemData.isShowExp && itemData.isShowExp.length > 0 ? this.colItemShowExps(itemData, item) : itemData.display === false ? false : true;
					} else {
						itemData['showExp'] = itemData['showExp'] || true;
					}
					if (itemData.formulaShow) {
						itemData['showExp'] = evaluatorTo(item, itemData.formulaShow);
						itemData['display'] = itemData['showExp']
					}
					itemData.valid = {
						column: itemData.column,
						valid: true,
						msg: '不能为空!'
					};
					this.specialCol.forEach(special => {
						if (special.column === itemData.column) {
							special.value ? (this.fieldModel[itemData.column] = special.value) : '';
							special.value ? (itemData['value'] = special.value) : '';
							special.disabled ? (itemData['disabled'] = special.disabled) : '';
							special.display ? (itemData['display'] = special.display) : '';
						}
					});
					return itemData;
				});
				this.allField.forEach(fileIf => {});
				console.log('0000000000000000', this.allField);
			}
		},
		onValChange(e) {
			const self = this;
			if (e.type === 'number') {
				this.fieldModel[e.column] = Number(e.value);
			} else {
				this.fieldModel[e.column] = e.value;
			}
			if (e.column === 'fwbm') {
				if (e.condition && Array.isArray(e.condition) && e.condition.length > 0 && e.condition[0].colName === e.condition[0].value) {
					e.condition.forEach(col => {
						this.fieldModel[col.value] = e.colData[col.value];
						this.$emit('changeFieldModel',this.fieldModel)
						self.allField.forEach((field,index)=>{
							if(field.column===col.value){
								field.value = e.colData[col.value]
								// self.$set(self.allField,index,field)
							}
						})
					});
				}
			}

			e.value = this.fieldModel[e.column];
			const fieldModel = JSON.parse(JSON.stringify(this.fieldModel));
			this.allField = this.allField.map((item, index) => {
				item.display = item.isShowExp && item.isShowExp.length > 0 ? this.colItemShowExps(item, this.fieldModel) : item.display === false ? false : true;
				if (item.column === e.column) {
					item.value = e.value;
				}
				return item;
			});
			this.allField.forEach(fileIf => {
				if (fileIf.formulaShow) {
					let isIfShow = evaluatorTo(fieldModel, fileIf.formulaShow);
					fileIf.display = isIfShow;
				}
			});
			// return
			if (e.bx_col_type === 'fk' && e.colData && typeof e.colData === 'object' && Array.isArray(e.colData) !== true && Object.keys(e.colData).length > 0) {
				//冗余
				this.allField.forEach(item => {
					if (item.redundant && typeof item.redundant === 'object' && item.redundant.dependField === e.column) {
						if (item.redundant.trigger === 'always') {
							item.value = e.colData[item.redundant.refedCol];
						} else if (item.redundant.trigger === 'isnull') {
							if (!item.value) {
								item.value = e.colData[item.redundant.refedCol];
							}
						}
						this.fieldModel[item.column] = item.value;
					}
				});
			}
			this.$emit('changeFieldModel',this.fieldModel)
			console.log('valueChange', e, this.fieldModel[e.column], this.fieldModel);
		},
		onValBlur(e) {
			console.log('e', e, this.fieldModel, this.fieldModel[e.column]);
			this.fieldModel[e.column] = e.value;
			const self = this;
			this.$emit('changeFieldModel',this.fieldModel)
			this.$emit('value-blur', e);
		},
		getDetailfieldModel() {
			return this.fieldModel;
		},
		getFieldModel() {
			console.log(this.fieldModel,'getFieldModel');
			let valid = 0;
			let showsNum = 0;
			this.allField.map((item, index) => {
				let valids = this.$refs.fitem[index].getValid();
				console.log('字段校验', valids,item);
				if (item.display) {
					showsNum++;
					if (valids.valid) {
						valid++;
					}
				}
			});
			console.log(valid, showsNum);
			if (valid === showsNum) {
				console.log('表单校验通过', showsNum, valid, this.fieldModel);
				let model = {};
				switch (this.pageType) {
					case 'update':
						for (let key in this.fieldModel) {
							if (this.oldFieldModel[key] !== this.fieldModel[key]) {
								model[key] = this.fieldModel[key];
							}
						}
						break;
					case 'add':
						for (let key in this.fieldModel) {
							if (this.fieldModel[key] === '' && key !== 'openid') {
								delete this.fieldModel[key];
							}
						}
						model = this.fieldModel;
						break;
					default:
						model = this.fieldModel;
						break;
				}
				console.log('this.fieldModel', this.fieldModel, model);
				if (Object.keys(model).length > 0) {
					return model;
				} else {
					uni.showToast({
						title: '没有需要提交的数据',
						icon: 'none'
					});
				}
			} else {
				console.log('表单校验失败', showsNum, valid, this.fieldModel);
				uni.showToast({
					title: '请填写完信息后，再尝试提交',
					icon: 'none'
				});
				return false;
			}
		},
		getCascaderVal(val) {
			if (val) {
				this.$emit('get-cascader-val', val);
			} else {
				this.$emit('get-cascader-val');
			}
		},
		onReset() {
			this.allField = this.deepClone(this.oldField);
			console.log(this.oldField, 'this.oldField');
			try {
				return true;
			} catch (e) {
				return false;
				//TODO handle the exception
			}
		}
	},
	watch: {
		fields: {
			handler: function(newval, old) {
				console.log('newval', newval, this.allField);
				this.getAllField();
			},
			deep: true
		},
		moreConfig: {
			handler: function(newval, old) {
				if (newval) {
					this.more_config = this.deepClone(newval);
				}
			},
			deep: true
		},
		fieldModel: {
			deep: true,
			handler(newVal, oldVal) {}
		}
	}
};
</script>

<style lang="scss" scoped>
	.form-box{
		display: flex;
		flex-wrap: wrap;
		.form-box-item{
			min-width: 50%;
		}
	}
</style>
