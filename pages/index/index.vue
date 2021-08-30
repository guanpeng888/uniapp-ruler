<template>
	<view class="ruler-wrap">
		<!-- 日期选择 -->
<!-- 		<calendar-select 
		v-model="showCalendar"
		:addOrRemoveData="addOrRemoveData"
		mode="range"
		:altPrice="dataPrice"
		@change="changeDate"></calendar-select> -->
		
		
		
		<view class="cur-value">肉类卡路里：{{foodkg}} -- {{kcal}}</view>
		<ZxRuler
		 CTRLID="heightRuler"
		 :min="100"
		 :max="230"
		 v-model="mainValue"
		 @onScrollChange="getRulerValue"></ZxRuler>
	</view>
</template>

<script>
	import calendarSelect from '../../components/calendar-select/calendar-select.vue'
	import ZxRuler from '../../components/ZxRuler.vue'
	export default {
		components:{
			ZxRuler,
			calendarSelect
		},
		data() {
			return {
				showCalendar: true,
				dataPrice: [ // 日历上的价格
					{
						date: '2021-9-23',
						price: 98
					},
					{
						date: '2021-9-28',
						price: 108
					},
					{
						date: '2021-9-29',
						price: 98
					}
				],
				addOrRemoveData: ['2021-08-23', '2021-08-27'],
				mainValue: 162,
				kcal: 0,
				foodkg: 0
			}
		},
		methods: {
			getRulerValue(result){
				this.foodkg = result;
				this.kcal = (result*8.2).toFixed(1);
			},
			// 获取选中的日期
			changeDate(data) {
				console.log(data)
			}
		},
	}
</script>

<style>
	.ruler-wrap{
		display: inline-block;
		margin: 30rpx;
		text-align: center;
	}
	.cur-value{
		text-align: center;
		font-size: 16px;
	}
</style>
